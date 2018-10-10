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
	#rm -rf /var/lib/apt/lists/* && \
	curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -&& \
	sudo apt-get install -y \
	nodejs \
	build-essential && \
	#curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash && \
	#sudo apt-get install -y \
	#git-lfs && \
	rm -rf /var/lib/apt/lists/*

# If we want to use the latest 7.1.x we can download it this way
# RUN curl ${FILES_LIFERAY}/private/ee/portal/snapshot-7.1.x-private/latest/liferay-portal-tomcat-7.1.x-private.7z --output liferay-portal-tomcat-7.1.x-private.7z && \
RUN curl ${FILES_LIFERAY}/private/ee/portal/snapshot-7.1.x/latest/liferay-portal-tomcat-7.1.x.7z --output liferay-portal-tomcat-7.1.x.7z

# # Or copying it from the host with passing along with the build context
# COPY liferay-portal-tomcat-7.1.x-private.7z .

# Copy the whole com-liferay-commerce repository
COPY . com-liferay-commerce/

# There are some necessary config files and Apio headless resource in the
# preparated branch (DXP-BIN) in com-liferay-commerce that we need to copy to the
# proper place in the image
RUN cd com-liferay-commerce/ && \
	export WORKING_BRANCH=`git branch | grep \* | cut -d ' ' -f2-` && \
	git reset --hard && \
	git checkout DXP-BIN && \
	cd /liferay && \
	# Move the files to the proper directory structure
	7z liferay-portal-tomcat-7.1.x.7z && \
	rm -rf liferay-portal-tomcat-7.1.x.7z && \
	mv liferay-portal-7.1.x bundles && \
	\
	# Remove the unwanted private modules (mobile detection) -> Bug in the portal private build
	#\
	#rm -rf /liferay/bundles/osgi/portal/com.liferay.portal.mobile*.jar && \
	# Copy the config files for Tomcat
	cp -f com-liferay-commerce/config/* bundles/tomcat/bin && \
	# Copy necessary Headless APIs to DXP
	cp com-liferay-commerce/modules/* bundles/osgi/modules && \
	# Reset to the original branch
	cd com-liferay-commerce && \
	git reset --hard && \
	git checkout ${WORKING_BRANCH} && \
	git clean -fdx

# Set up gradle global properties for the test framework
RUN mkdir -p /root/.gradle && \
	echo "app.server.tomcat.dir=/liferay/bundles/tomcat\n\
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
	cd /liferay/com-liferay-commerce/commerce-data-integration/commerce-data-integration-apio-end-to-end-test && \
	./../../gradlew setUpTestableTomcat && \
	\
	# Download test compile dependencies in advance
	\
	./../../gradlew clean compileTestIntegrationJava

# Remove repo as we need will need the latest state that will be come from the build context
RUN rm -rf /liferay/com-liferay-commerce