SHELL := /bin/bash
.PHONY: publish layers docker-images

# Publish the layers on AWS Lambda
publish: layers
	cd layers ; php publish.php

# Build the layers
layers: export/console.zip export/php-73.zip export/php-74.zip export/php-80.zip export/php-81.zip export/php-82.zip export/php-73-fpm.zip export/php-74-fpm.zip export/php-80-fpm.zip export/php-81-fpm.zip export/php-82-fpm.zip

# The PHP runtimes
export/php%.zip: docker-images
	PHP_VERSION=$$(echo $@ | cut -d'/' -f 2 | cut -d'.' -f 1);\
	rm -f $@;\
	cd export ; \
	set -e ; \
	rm -rf opt ; \
	CID=$$(docker create --entrypoint=scratch bref/$$PHP_VERSION:latest) ; \
	docker cp $${CID}:/opt . ; \
	docker rm $${CID} ; \
	cd opt ; \
	zip -qq -y -r - {*,.[!.]*} > ../$$PHP_VERSION.zip
	cd ../ ; \
	rm -rf opt ;

# The console runtime
export/console.zip: layers/console/bootstrap
	rm -f export/console.zip
	cd layers/console && zip ../../export/console.zip bootstrap breftoolbox.php

# Build Docker images
docker-images:
	# Build the base environment (without PHP)
	cd base ; docker build --file base.Dockerfile -t bref/tmp/step-1/build-environment .
	# Build the `bref/build-php-XX` images
	# (build only the first `FROM` section of the Dockerfile)
	cd base ; docker build --file php-73.Dockerfile -t bref/build-php-73 --target build-environment .
	cd base ; docker build --file php-74.Dockerfile -t bref/build-php-74 --target build-environment .
	cd base ; docker build --file php-80.Dockerfile -t bref/build-php-80 --target build-environment .
	cd base ; docker build --file php-81.Dockerfile -t bref/build-php-81 --target build-environment .
	cd base ; docker build --file php-82.Dockerfile -t bref/build-php-82 --target build-environment .
	# Build the whole Dockerfile to generate the cleaned images that will be used in the next step
	cd base ; docker build --file php-73.Dockerfile -t bref/tmp/cleaned-build-php-73 .
	cd base ; docker build --file php-74.Dockerfile -t bref/tmp/cleaned-build-php-74 .
	cd base ; docker build --file php-80.Dockerfile -t bref/tmp/cleaned-build-php-80 .
	cd base ; docker build --file php-81.Dockerfile -t bref/tmp/cleaned-build-php-81 .
	cd base ; docker build --file php-82.Dockerfile -t bref/tmp/cleaned-build-php-82 .
	# - function
	cd layers/function ; docker build -t bref/php-73 --build-arg PHP_VERSION=73 .
	cd layers/function ; docker build -t bref/php-74 --build-arg PHP_VERSION=74 .
	cd layers/function ; docker build -t bref/php-80 --build-arg PHP_VERSION=80 .
	cd layers/function ; docker build -t bref/php-81 --build-arg PHP_VERSION=81 .
	cd layers/function ; docker build -t bref/php-82 --build-arg PHP_VERSION=82 .
	# - fpm
	cd layers/fpm ; docker build -t bref/php-73-fpm --build-arg PHP_VERSION=73 .
	cd layers/fpm ; docker build -t bref/php-74-fpm --build-arg PHP_VERSION=74 .
	cd layers/fpm ; docker build -t bref/php-80-fpm --build-arg PHP_VERSION=80 .
	cd layers/fpm ; docker build -t bref/php-81-fpm --build-arg PHP_VERSION=81 .
	cd layers/fpm ; docker build -t bref/php-82-fpm --build-arg PHP_VERSION=82 .
	# - console
	cd layers/console ; docker build -t bref/php-73-console --build-arg PHP_VERSION=73 .
	cd layers/console ; docker build -t bref/php-74-console --build-arg PHP_VERSION=74 .
	cd layers/console ; docker build -t bref/php-80-console --build-arg PHP_VERSION=80 .
	cd layers/console ; docker build -t bref/php-81-console --build-arg PHP_VERSION=81 .
	cd layers/console ; docker build -t bref/php-82-console --build-arg PHP_VERSION=82 .
	# Other Docker images
	cd layers/fpm-dev ; docker build -t bref/php-73-fpm-dev --build-arg PHP_VERSION=73 .
	cd layers/fpm-dev ; docker build -t bref/php-74-fpm-dev --build-arg PHP_VERSION=74 .
	cd layers/fpm-dev ; docker build -t bref/php-80-fpm-dev --build-arg PHP_VERSION=80 .
	cd layers/fpm-dev ; docker build -t bref/php-81-fpm-dev --build-arg PHP_VERSION=81 .
	# cd layers/fpm-dev ; docker build -t bref/php-82-fpm-dev --build-arg PHP_VERSION=82 .
	cd layers/web; docker build -t bref/fpm-dev-gateway .
	# Run tests
	php layers/tests.php
