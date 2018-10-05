FROM openjdk:8-jdk-slim

LABEL maintainer="zoltan.takacs@liferay.com"

WORKDIR /testResults

RUN echo "#!/bin/bash\n\
echo "Started:" `date`\n"\
> test.sh

RUN ls

RUN chmod +x test.sh

ENTRYPOINT ./test.sh