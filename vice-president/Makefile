DATE    = $(shell date +%Y%m%d%H%M)
IMAGE   ?= sapcc/vice-president
VERSION = v$(DATE)
GOOS    ?= $(shell go env | grep GOOS | cut -d'"' -f2)
BINARY  := vice-president

LDFLAGS := -X github.com/sapcc/kubernetes-operators/vice-president/pkg/president.VERSION=$(VERSION)
GOFLAGS := -ldflags "$(LDFLAGS)"

SRCDIRS  := cmd pkg
PACKAGES := $(shell find $(SRCDIRS) -type d)
GOFILES  := $(addsuffix /*.go,$(PACKAGES))
GOFILES  := $(wildcard $(GOFILES))

GLIDE := $(shell command -v glide 2> /dev/null)

.PHONY: all clean

all: bin/$(GOOS)/$(BINARY)

bin/%/$(BINARY): $(GOFILES) Makefile
	GOOS=$* GOARCH=amd64 go build $(GOFLAGS) -v -i -o bin/$*/$(BINARY) ./cmd

build: bin/linux/$(BINARY)
	docker build -t $(IMAGE):$(VERSION) .

static-check:
	@if s="$$(gofmt -s -l *.go pkg 2>/dev/null)"                            && test -n "$$s"; then printf ' => %s\n%s\n' gofmt  "$$s"; false; fi
	@if s="$$(golint . && find pkg -type d -exec golint {} \; 2>/dev/null)" && test -n "$$s"; then printf ' => %s\n%s\n' golint "$$s"; false; fi

tests: all static-check
	go test -v github.com/sapcc/kubernetes-operators/vice-president/pkg/president

push: build
	docker push $(IMAGE):$(VERSION)

clean:
	rm -rf bin/*

vendor:
	ifndef GLIDE
		echo "Cannot vendor. Glide is not installed. Please do so now."
	else
		glide install --strip-vendor --strip-vcs && glide update --strip-vendor --strip-vcs
	endif
