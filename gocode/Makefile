ROOT=..
SBE_TOOL_VERSION=$(shell cat $(ROOT)/version.txt)
SBE_JAR=$(ROOT)/sbe-all/build/libs/sbe-all-${SBE_TOOL_VERSION}.jar
# FIXME: Go doesn't like relative paths so us a gnumake extension for now
GOPATH=$(realpath $(ROOT)/gocode)


# Convenience during development
#SAVE_FORMAT=mkdir -p fmt && cp *.go fmt &&
SAVE_FORMAT=

all: example test # bench


# This target is used to build golang files using parent gradle
# script's invocation of sbe-tool in case it needs to be run again. We
# use this one output file to check that dependency as it's simple
DEP=src/simple/SbeMarshalling.go
$(DEP): $(SBE_JAR)
	(cd ..; gradlew generateGolangCodecs)
	(export GOPATH=$(GOPATH) && \
	cd src/simple && \
	go build && \
	$(SAVE_FORMAT) \
	go fmt && \
	go test)

# Wil regenerate codecs by removing dependencies
clean:
	rm -f src/*/SbeMarshalling.go src/*/*/SbeMarshalling.go

# Example code and benchmarking 
example: src/example-schema/example-schema
src/example-schema/example-schema: $(DEP)
	(export GOPATH=$(GOPATH) && \
	cd src/example-schema && \
	go build && \
	go fmt && \
	./example-schema)

bench: $(DEP) src/example-schema/example-schema.test
	(export GOPATH=$(GOPATH) && \
        cd src/example-schema && \
	go test --bench . -cpuprofile=cpu.out && \
	go tool pprof -text example-schema.test cpu.out)

src/baseline/SbeMarshalling.go: $(DEP)
	$(GOPATH)/sbe-tool -d src -s $(EXAMPLES_SCHEMA)
	(export GOPATH=$(GOPATH) && \
        cd src/baseline && \
	go build && \
	go fmt && \
	go test && \
	go install)

# The first set does a make install as there is a test that uses
# multiple packages and needs them in GOPATH The second set work in
# src/foo, and the third need a GOPATH addition as for Java and C++
# they geenrate into the same directory but golang doesn't allow that
test: $(DEP)
	(export GOPATH=$(GOPATH) && \
		(for t in baseline extension extension2; do \
		export GOPATH=$(GOPATH) && \
	        cd $(GOPATH)/src/$$t && \
		go build && \
		go fmt && \
		go test && \
		go install \
		;done))
	(export GOPATH=$(GOPATH) && \
		(for t in mktdata group_with_data group_with_data_extension composite_element composite simple; do \
		cd $(GOPATH)/src/$$t && \
		go build && \
		go fmt && \
		go test \
		;done))
	(for t in vardata group basic; do \
		export GOPATH=$(GOPATH)/$$t && \
		cd $(GOPATH)/src/$$t/'SBE tests' && \
		go build && \
		go fmt && \
		go test \
		;done)
