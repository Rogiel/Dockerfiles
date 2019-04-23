TAG=rogiel/cpp-build

all: build

build: Makefile Dockerfile
	docker build -t $(TAG) .
	@echo "Done!  Use docker run $(TAG) to run"

push:
	docker push $(TAG)
