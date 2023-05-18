FROM	alpine:latest

ARG	REPO_URL
ARG	MAIN_SRC_DIR
ARG	TEST_SRC_DIR

ENV	REPO_URL=${REPO_URL}
ENV	MAIN_SRC_DIR=${MAIN_SRC_DIR}
ENV	TEST_SRC_DIR=${TEST_SRC_DIR}

RUN	apk add --no-cache git openjdk11 && \
	mkdir /app 

WORKDIR	/app

RUN	cd /app && \
	git clone -o origin ${REPO_URL} . && \
	mkdir -p config/checkstyle

CMD	["./gradlew", "check"]

