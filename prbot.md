## Introduction

This is the design document of the PR bot in my GSoC 2017 project.

Status: Draft

## Architecture

The PR bot is written in Go and makes heavy use of goroutines.

We don't have much traffic of PRs so every role dispatches a new goroutine when request comes in.

## Roles

### GitHub Webhook Listener [core]

* From: HTTP Server
* To: PR Handler

#### Description

When the program starts a GithubWebhookListener struct is created and the HTTP request is passed to its method `Do(w http.ResponseWriter, r *http.Request)`. If the request is vaild, WG counter is incremented, then a goroutine is started to process the event.

#### Functions

* Filter webhook events
* Verify X-Hub-Signature
* Log webhook events

### CI Bot Listener [core]

* From: HTTP Server
* To: Travis Log Handler

#### Description

Gather and process information from the CI bot (maybe in FlatBuffers).

### PR Handler [core]

* From: GitHub Webhook Listener
* To: Mailer

#### Description

When a PullRequestEvent is received, `Do(prID int)` is called and more information will be fetched with [go-github](https://github.com/google/go-github). Start a goroutine to send mails if global config permits.

#### Functions

* Add labels
* Comment to mention maintainer

### Mailer [extra]

* From: PR Handler
* To: Mailer

#### Description

Mail maintainer if a PR is created and we don't have the maintainer's GitHub handle.

#### Functions

* Mail maintainer
* CC mailing list

### Review Handler [maybe]

#### Description

When a PullRequestReviewEvent is received, detect if the reviewer is the maintainer and add corresponding labels.

### Maintainer Timeout Handler [maybe]

#### Description

Runs daily and query the database for `maintainer:timeout` PRs. If no comment from maintainer exist in the PR, labels are changed accordingly and database is updated.

### Travis Log Handler [maybe, fallback]

* From: CI Bot Listener, Build Timer

#### Description

Parse log when a Travis build finishes.

## Helpers (synchronous, stateless)

### Maintainer Lookup

#### Description

Queries the database for maintainer info.

## Shutdown

The program maintains a [WaitGroup](https://golang.org/pkg/sync/#WaitGroup) counting the number of goroutines called - finished.

After calling [Shutdown](https://golang.org/pkg/net/http/#Server.Shutdown) to shut down the HTTP server, the program waits until no more goroutines are executing or being called.

Receiving two SIGINT would force immediate shutdown.

## Considerations

* Use Go's stdlib whenever possible (security, maintainability)
* Secrets read from environment variables
* Independent of CI provider (? if not needed, can get ECDSA public key from log and verify messages)
* CI bot can't be trusted

## Appendix

### Labels

* `maintainer:none` `maintainer:open` `maintainer:waiting` `maintainer:approved` (also when maintainer is submitter) `maintainer:timeout`
* `type:bugfix` `type:update` `type:enhancement` `type:submission`
* `priority:high` `priority:security`
