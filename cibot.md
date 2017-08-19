## Introduction

This is the design document of the Travis (CI) bot in my GSoC 2017 project.

## Overview

The CI bot is written in Go and controls the build process on Travis. It sends data to the PR bot.

## Functions

1. List subports
2. `port lint` test
3. `port -d install` test
4. Send data to PR bot
5. Execute custom commands specified in PR body (maybe)

All functions are done in separate goroutines.

## Log

The CI bot controls stdout and enforces a machine-parsable format (MIME-like?). This is done in a separate goroutine. Raw log is written to disk first (considering huge logs eating RAM) by other goroutines, and then an io.Reader is passed via a channel (Go) to the log goroutine.

MIME part names: `port-*-dep-{summary,install}, port-*-install, port-*-lint, ecdsa-pubkey`

## Interaction with CI Bot

1. The CI bot generates MIME multipart logs and the PR bot parses these logs from Travis API.
2. The CI bot generates an ECDSA key pair on start and prints the public key on Travis log. While testing ports, the bot attempts handshake with the PR bot by signing the salt PR bot provided (TCP or HTTP?). The PR bot would grab the public key from Travis logs and verify the signature.

This design is aimed for traceability, we can find the exact GitHub user who submitted a malicious PR.
