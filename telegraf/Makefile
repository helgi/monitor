SHELL = /bin/bash

DOCKER_HOST = $(shell echo $$DOCKER_HOST)
BUILD_TAG ?= git-$(shell git rev-parse --short HEAD)
SHORT_NAME ?= telegraf
DEIS_REGISTRY ?= ${DEV_REGISTRY}
IMAGE_PREFIX ?= deis

include versioning.mk

build: docker-build
push: docker-push
install: kube-install
uninstall: kube-delete
upgrade: kube-update

docker-build:
	docker build -t ${IMAGE} rootfs
	docker tag ${IMAGE} ${MUTABLE_IMAGE}

clean: check-docker
	docker rmi $(IMAGE)

update-manifests:
	sed 's#\(image:\) .*#\1 $(IMAGE)#' manifests/deis-monitor-telegraf-daemon.yaml > manifests/deis-monitor-telegraf-daemon.tmp.yaml

kube-install: update-manifests
	kubectl create -f manifests/deis-monitor-telegraf-service-account.yaml
	kubectl create -f manifests/deis-monitor-telegraf-daemon.yaml

kube-delete:
	kubectl delete -f manifests/deis-monitor-telegraf-service-account.yaml
	kubectl delete -f manifests/deis-monitor-telegraf-daemon.yaml

kube-update: update-manifests
	kubectl delete -f manifests/deis-monitor-telegraf-daemon.tmp.yaml
	kubectl create -f manifests/deis-monitor-telegraf-daemon.tmp.yaml
	