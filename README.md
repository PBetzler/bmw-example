# cifuzz bazel example

This is a simple bazel based project, already configured with
**cifuzz**. It should quickly produce a finding, but slow enough to
see the progress of the fuzzer.

To start make sure you installed **cifuzz** according to the
main [README](../../README.md).

You can start the fuzzing with

```bash
cifuzz run //src:explore_me_fuzz_test -v
```

## Create regression test

After you have discovered a finding, you may want to include this as
part of a regression test. To replay findings from the
`src/explore_me_fuzz_test_inputs` directory:

```bash
bazel test --config=cifuzz-replay //src:explore_me_fuzz_test --test_output=streamed
```

Note that this requires these lines in your `.bazelrc`:

```bash
# Replay cifuzz findings (C/C++ only)
build:cifuzz-replay --@rules_fuzzing//fuzzing:cc_engine_sanitizer=asan-ubsan
build:cifuzz-replay --compilation_mode=opt
build:cifuzz-replay --copt=-g
build:cifuzz-replay --copt=-U_FORTIFY_SOURCE
build:cifuzz-replay --test_env=UBSAN_OPTIONS=halt_on_error=1
```


## BMW integration:

Candidates selection test commands: 
```bash
bazel run //:refresh_compile_commands
cifuzz candidates --build-dir .
```

LLM powered create test commands:
```bash
CIFUZZ_LLM_API_URL=... \
CIFUZZ_LLM_API_TOKEN=... \
CIFUZZ_LLM_API_TYPE=BMW \
CIFUZZ_LLM_API_HEADER_Authorization="Bearer [WEBEAM_ACCESS_TOKEN]" \
cifuzz create exploreMe --build-dir .
```

### LLM Configuration

Currently, configuration is done only via environment variables.

- CIFUZZ_LLM_API_URL - Base URL for the LLM API server.
- CIFUZZ_LLM_API_TOKEN - API token when talking to the LLM API.
  In the BMW LLM client this will be set as the `x-apikey` header.
- CIFUZZ_LLM_API_TYPE - Supported: BMW, OPEN_AI, AZURE, AZURE_AD,
  CLOUDFLARE_AZURE.
- CIFUZZ_LLM_MODEL - LLM model to use.
- CIFUZZ_LLM_TEMPERATURE - Temperature setting for chat completion.
- CIFUZZ_LLM_MAX_TOKENS - Maximum number of tokens for a single chat completion
  request.
- CIFUZZ_LLM_API_HEADER_<header-name> - Additional headers to add to HTTP
  requests. Multiple possible. "\_" in <header-name> is replaced by "-" in the
  header.
- CIFUZZ_LLM_API_POLL_INTERVAL_MILLIS - Poll interval in milliseconds (only BMW
  client).

### Trust custom CA

The `SSL_CERT_FILE` environment variable can be set to point to a pem file that
will be used to verify SSL connections with the LLM API server (see
https://go.dev/src/crypto/x509/root_unix.go for details).


## Devcontainer setup:

To use the devcontainer environment you need to export your cifuzz download token to a environment var called "CIFUZZ_CREDENTIALS" like `export CIFUZZ_CREDENTIALS=[my_token]`.