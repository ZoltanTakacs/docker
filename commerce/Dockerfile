FROM openjdk:8u181-jdk as builder

LABEL maintainer="zoltan.takacs@liferay.com"

WORKDIR /liferay

ENV FILES_LIFERAY=http://files.liferay.com

RUN apt-get update && apt-get install -y --no-install-recommends \
	curl \
	git \
	mc \
	screen \
	sudo \
	p7zip-full && \
	rm -rf /var/lib/apt/lists/* && \
	curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -&& \
	sudo apt-get install -y \
	nodejs \
	build-essential && \
	curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash && \
	sudo apt-get install -y \
	git-lfs && \
	rm -rf /var/lib/apt/lists/*

# # If we want to use the latest 7.1.x-private we can download it this way
# RUN curl ${FILES_LIFERAY}/private/ee/portal/snapshot-7.1.x-private/latest/liferay-portal-tomcat-7.1.x-private.7z --output liferay-portal-tomcat-7.1.x-private.7z && \

# # Or copying it from the host with passing along with the build context
# COPY liferay-portal-tomcat-7.1.x-private.7z .

# The current Dockerfile present in the commerce-private repo which is the
# current build context, copy this to the image
COPY . com-liferay-commerce-private/

# The 7.1.x-private bundle is committed to a preparated branch (DXP-BIN) in
# commerce-private repo so we don't need to download it from elsewhere
RUN cd com-liferay-commerce-private/ && \
	git checkout DXP-BIN && \
	cd /liferay && \
	# Move the files to the proper directory structure
	cp com-liferay-commerce-private/portal/liferay-portal-tomcat-7.1.x-private.7z liferay-portal-tomcat-7.1.x-private.7z && \
	7z x liferay-portal-tomcat-7.1.x-private.7z && \
	rm -rf liferay-portal-tomcat-7.1.x-private.7z && \
	mv liferay-portal-7.1.x-private bundles && \
	\
	# Remove the unwanted private modules (mobile detection) -> Bug in the portal private build
	\
	rm -rf /liferay/bundles/osgi/portal/com.liferay.portal.mobile*.jar && \
	# Copy the config files for Tomcat
	cp -f com-liferay-commerce-private/config/* bundles/tomcat/bin && \
	# Copy necessary Headless APIs to DXP
	cp com-liferay-commerce-private/modules/* bundles/osgi/modules && \
	# Reset to the original branch
	cd com-liferay-commerce-private && \
	git reset --hard && \
	git checkout COMMERCE-628 && \
	git clean -fdx

# Set up gradle global properties for the test framework
RUN echo "app.server.tomcat.dir=/liferay/bundles/tomcat\n\
liferay.home=/liferay/bundles\n"\
> /root/.gradle/gradle.properties

ENV CATALINA_OPTS="${CATALINA_OPTS} -server" \
\
GRADLE_OPTS="-server -Xmx3g -Xms3g -server -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -Dorg.gradle.daemon=false" \
\
# For debugging
\
JPDA_ADDRESS=0.0.0.0:8000 \
\
LIFERAY_HOME=/liferay/bundles \
LIFERAY_LIFERAY_PERIOD_HOME=/liferay/bundles \
\
# Default values
\
LIFERAY_ADMIN_PERIOD_EMAIL_PERIOD_FROM_PERIOD_ADDRESS=test@liferay.com \
LIFERAY_ADMIN_PERIOD_EMAIL_PERIOD_FROM_PERIOD_NAME=Test\ Test \
LIFERAY_COMPANY_PERIOD_DEFAULT_PERIOD_LOCALE=en_US \
LIFERAY_COMPANY_PERIOD_DEFAULT_PERIOD_WEB_PERIOD_ID=liferay.com \
LIFERAY_DEFAULT_PERIOD_ADMIN_PERIOD_EMAIL_PERIOD_ADDRESS_PERIOD_PREFIX=test \
\
# Setup wizard related properties
\
LIFERAY_SETUP_PERIOD_WIZARD_PERIOD_ADD_PERIOD_SAMPLE_PERIOD_DATA=true \
LIFERAY_SETUP_PERIOD_WIZARD_PERIOD_ENABLED=false \
LIFERAY_TERMS_PERIOD_OF_PERIOD_USE_PERIOD_REQUIRED=false \
LIFERAY_USERS_PERIOD_REMINDER_PERIOD_QUERIES_PERIOD_CUSTOM_PERIOD_QUESTION_PERIOD_ENABLED=false \
LIFERAY_USERS_PERIOD_REMINDER_PERIOD_QUERIES_PERIOD_ENABLE=false \
\
# Modules framework validator to false to speed up the bundle startup
\
LIFERAY_MODULE_PERIOD_FRAMEWORK_PERIOD_PROPERTIES_PERIOD_LPKG_PERIOD_INDEX_PERIOD_VALIDATOR_PERIOD_ENABLED=false

# Expose default ports: jpda, shutdown, ajp, http, jmx, https, osgi
EXPOSE 8000 8005 8009 8080 8099 8443 11311

# Setup integration test framework
RUN echo "Value of CATALINA_OPTS: ${CATALINA_OPTS}" && \
	cd /liferay/com-liferay-commerce-private/commerce-data-integration/commerce-data-integration-apio-end-to-end-test && \
	./../../gradlew setUpTestableTomcat

### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ###
### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ###
### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ###

# Compile and deploy commerce public modules to DXP
RUN cd /liferay && \
	git clone https://github.com/liferay/com-liferay-commerce.git --depth 1 --single-branch --branch 7.1.x && \
	cd com-liferay-commerce && \
	./gradlew deploy -Dbuild.profile=portal -PnodeDownload=false --parallel

# Compile and deploy commerce private modules to DXP
RUN cd /liferay/com-liferay-commerce-private && \
	./gradlew deploy -Dbuild.profile=portal -PnodeDownload=false --parallel

# Add volume for the document library
# VOLUME [ "/liferay/bundles/data/document_library" ]

WORKDIR /liferay/com-liferay-commerce-private/commerce-data-integration/commerce-data-integration-apio-end-to-end-test
CMD ./../../gradlew clean testIntegration

# CMD ./liferay/bundles/tomcat/bin/catalina.sh jpda run