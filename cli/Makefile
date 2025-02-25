PWD := $(shell pwd)
PARENT_DIR := $(shell dirname $(PWD))
VERSION ?= $(shell ../scripts/get-version.sh)
PLUGINS_BRANCH ?= master
OS ?= linux darwin windows
OUTPUT_DIR := _output

HAS_GOX := $(shell command -v gox;)
HAS_GINKGO := $(shell command -v ginkgo;)

ifeq (${CI}, true)
	CLI_EXT_BIN_DIR := "/build/_cli_ext_output"
else
	CLI_EXT_BIN_DIR := "../cli-ext/bin"
endif

.PHONY: deps
# get dependencies
deps:
	go mod download

# build cli locally, for all given platform/arch
.PHONY: build
build: export CGO_ENABLED=0
build: copy-cli-ext
build:
ifndef HAS_GOX
	cd ~ && go get github.com/mitchellh/gox && cd -
endif
	gox -ldflags '-X github.com/hasura/graphql-engine/cli/v2/version.BuildVersion=$(VERSION) -X github.com/hasura/graphql-engine/cli/v2/plugins.IndexBranchRef=$(PLUGINS_BRANCH) -s -w -extldflags "-static"' \
	-rebuild \
	-os="$(OS)" \
	-arch="amd64" \
	-output="$(OUTPUT_DIR)/$(VERSION)/cli-hasura-{{.OS}}-{{.Arch}}" \
	./cmd/hasura/

# compress
.PHONY: compress
compress:
	ls $(OUTPUT_DIR)/$(VERSION)/cli-hasura-* | grep -v darwin | xargs upx

# to be executed in circle-ci only
ci-copy-binary:
	mkdir -p /build/_cli_output/binaries
	cp $(OUTPUT_DIR)/$(VERSION)/cli-hasura-* /build/_cli_output/binaries
	echo "$(VERSION)" > /build/_cli_output/version.txt

.PHONY: build-cli-ext
build-cli-ext:
	cd ../cli-ext && make deps && make build

.PHONY: clean-cli-ext
clean-cli-ext:
	rm -rf ./internal/cliext/bin

.PHONY: copy-cli-ext
copy-cli-ext: clean-cli-ext
	cp -r $(CLI_EXT_BIN_DIR) ./internal/cliext/bin

# run tests
.PHONY: test
test: copy-cli-ext
	if [ -z "${HASURA_TEST_CLI_HGE_DOCKER_IMAGE}" ]; then echo "Please set HASURA_TEST_CLI_HGE_DOCKER_IMAGE env for populating testutil.HasuraDockerImage and run test"; exit 1; fi
	go test -ldflags "-X github.com/hasura/graphql-engine/cli/v2/version.BuildVersion=$(VERSION)" -v -tags="${TEST_TAGS}" `go list ./... | grep -v integration_test | grep -v commands`
integration_tests_config_v3: copy-cli-ext
	if [ -z "${HASURA_TEST_CLI_HGE_DOCKER_IMAGE}" ]; then echo "Please set HASURA_TEST_CLI_HGE_DOCKER_IMAGE env for populating testutil.HasuraDockerImage and run test"; exit 1; fi
	go test -ldflags "-X github.com/hasura/graphql-engine/cli/v2/version.BuildVersion=$(VERSION)" -v -tags="${TEST_TAGS}" -run Commands/config=v3 ./integration_test
integration_tests_config_v2: copy-cli-ext
	if [ -z "${HASURA_TEST_CLI_HGE_DOCKER_IMAGE}" ]; then echo "Please set HASURA_TEST_CLI_HGE_DOCKER_IMAGE env for populating testutil.HasuraDockerImage and run test"; exit 1; fi
	go test -ldflags "-X github.com/hasura/graphql-engine/cli/v2/version.BuildVersion=$(VERSION)" -v -tags="${TEST_TAGS}" -run Commands/config=v2 ./integration_test
test-e2e:
ifndef HAS_GINKGO
	cd ~ && go get github.com/onsi/ginkgo/ginkgo && cd -
endif
	cd commands && ginkgo -p -v --failFast

test-all: test integration_tests_config_v2 integration_tests_config_v3 test-e2e
# clean the output directory
.PHONY: clean
clean:
	rm -rf "$(OUTPUT_DIR)"

.PHONY: all
all: deps test build

# build cli inside a docker container
all-in-docker:
	docker build -t hasura-graphql-cli-builder -f build/builder.dockerfile build
	docker run --rm -it \
	-v $(PARENT_DIR):/go/src/github.com/hasura/graphql-engine \
	hasura-graphql-cli-builder \
	make all
