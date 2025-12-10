ROOT_DIR ?= $(dir $(realpath $(lastword $(MAKEFILE_LIST))))
POLICY_DIR := $(notdir $(patsubst %/,%,$(ROOT_DIR)))
SOURCE_FILES := $(shell find . -type f -name '*.go')
VERSION ?= $(shell git describe | cut -c2-)

policy.wasm: $(SOURCE_FILES) 
# CI is a environment variable set in the GitHub Actions workflow.
# If we are running in CI, we assume TinyGo is already installed.
ifeq ($(CI),true)
	tinygo build -o policy.wasm -target=wasi -no-debug .
else
	docker run \
		--rm \
		-e GOFLAGS="-buildvcs=false" \
		-v $(ROOT_DIR)../:/src \
		-w /src tinygo/tinygo:0.39.0 \
		tinygo build -o $(POLICY_DIR)/policy.wasm -target=wasi -no-debug ./$(POLICY_DIR)
endif


annotated-policy.wasm: policy.wasm metadata.yml
	kwctl annotate -m metadata.yml -u README.md -o annotated-policy.wasm policy.wasm

.PHONY: test
test:
	go test -v

.PHONY: e2e-tests
e2e-tests: annotated-policy.wasm
	bats e2e.bats

.PHONY: lint
lint:
	go vet ./...
	golangci-lint run

.PHONY: clean
clean:
	go clean
	rm -f policy.wasm annotated-policy.wasm
