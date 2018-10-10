FROM devxr/commerce:latest

LABEL maintainer="zoltan.takacs@liferay.com"

WORKDIR /liferay

# The current Dockerfile present in the com-liferay-commerce repo which is the
# current build context, copy this to the image
COPY . com-liferay-commerce/

# Checkout the branch we need to work with and compile + deploy it to the portal
RUN cd com-liferay-commerce/ && \
	git reset --hard && \
	git clean -fdx && \
	./gradlew deploy -Dbuild.profile=portal -PnodeDownload=false --parallel

WORKDIR /liferay/com-liferay-commerce/commerce-data-integration/commerce-data-integration-apio-end-to-end-test
CMD ./../../gradlew clean testIntegration

# CMD ./liferay/bundles/tomcat-9.0.10/bin/catalina.sh jpda run