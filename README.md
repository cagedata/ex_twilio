ExTwilio
========
[![Hex.pm](https://img.shields.io/hexpm/v/ex_twilio.svg)](https://hex.pm/packages/ex_twilio)
[![Build Status](https://travis-ci.org/danielberkompas/ex_twilio.svg?branch=master)](https://travis-ci.org/danielberkompas/ex_twilio)
[![Inline docs](http://inch-ci.org/github/danielberkompas/ex_twilio.svg?branch=master)](http://inch-ci.org/github/danielberkompas/ex_twilio)

## Installation

ExTwilio is currently beta software. You can install it from Hex:

```elixir
def deps do
  [
    {:ibrowse, github: "cmullaparthi/ibrowse", tag: "v4.1.2"},
    {:ex_twilio, "~> 0.1.3"}
  ]
end
```

Or from Github:

```elixir
def deps do
  [
    {:ibrowse, github: "cmullaparthi/ibrowse", tag: "v4.1.2"},
    {:ex_twilio, github: "danielberkompas/ex_twilio"}
  ]
end
```

and run $ mix deps.get. Now, list the :ex_twilio application as your application dependency:

```elixir
def application do
  [applications: [:ex_twilio]]
end
```



## Configuration

You will need to set the following configuration variables in your 
`config/config.exs` file:

```elixir
use Mix.Config

config :ex_twilio, account_sid: System.get_env("TWILIO_ACCOUNT_SID"),
                   auth_token:  System.get_env("TWILIO_AUTH_TOKEN")
```

For security, I recommend that you use environment variables rather than hard
coding your account credentials. If you don't already have an environment
variable manager, you can create a `.env` file in your project with the
following content:

```bash
export TWILIO_ACCOUNT_SID=<account sid here>
export TWILIO_AUTH_TOKEN=<auth token>
```

Then, just be sure to run `source .env` in your shell before compiling your
project.

### Multiple Environments
If you want to use different Twilio credentials for different environments, then
create separate Mix configuration files for each environment. To do this, change
`config/config.exs` to look like this:

```elixir
# config/config.exs

use Mix.Config

# shared configuration for all environments here ...

import_config "#{Mix.env}.exs"
```

Then, create a `config/#{environment_name}.exs` file for each environment. You
can then set the `config :ex_twilio` variables differently in each file.

## Usage

ExTwilio comes with module for each supported Twilio API resource. For example,
the "Call" resource is accessible through the `ExTwilio.Call` module. Depending
on what the underlying API supports, a resource module may have the following
methods:

| Method            | Description                                                       |
| ----------------- | ----------------------------------------------------------------- |
| **all**           | Eager load all of the resource items on all pages. Use with care! |
| **stream**        | Create a Stream of all the items. Use like any Stream.            |
| **find**          | Find a resource given its SID.                                    |
| **create**        | Create a resource.                                                |
| **update**        | Update a resource.                                                |
| **destroy**       | Destroy a resource.                                               |

Resource modules may contain their own custom methods. If the underlying API
endpoint does not support an action, the related method will _not_ be available
on that module.

### Supported Endpoints

ExTwilio currently supports the following Twilio endpoints:

- [Account](https://www.twilio.com/docs/api/2010-04-01/rest/account). Including SubAccounts.
- [Address](https://www.twilio.com/docs/api/2010-04-01/rest/addresses)
    - [DependentPhoneNumber](https://www.twilio.com/docs/api/2010-04-01/rest/addresses#instance-subresources)
- [Application](https://www.twilio.com/docs/api/2010-04-01/rest/applications)
- [AuthorizedConnectApp](https://www.twilio.com/docs/api/2010-04-01/rest/authorized-connect-apps)
- [AvailablePhoneNumber](https://www.twilio.com/docs/api/2010-04-01/rest/available-phone-numbers)
- [Call](https://www.twilio.com/docs/api/2010-04-01/rest/call)
    - [Feedback](https://www.twilio.com/docs/api/2010-04-01/rest/call-feedback)
- [Conference](https://www.twilio.com/docs/api/2010-04-01/rest/conference)
    - [Participant](https://www.twilio.com/docs/api/2010-04-01/rest/participant)
- [ConnectApp](https://www.twilio.com/docs/api/2010-04-01/rest/connect-apps)
- [IncomingPhoneNumber](https://www.twilio.com/docs/api/2010-04-01/rest/incoming-phone-numbers)
- [Message](https://www.twilio.com/docs/api/2010-04-01/rest/message)
    - [Media](https://www.twilio.com/docs/api/2010-04-01/rest/media)
- [Notification](https://www.twilio.com/docs/api/2010-04-01/rest/notification)
- [OutgoingCallerId](https://www.twilio.com/docs/api/2010-04-01/rest/outgoing-caller-ids)
- [Queue](https://www.twilio.com/docs/api/2010-04-01/rest/queue)
    - [Member](https://www.twilio.com/docs/api/2010-04-01/rest/member)
- [Recording](https://www.twilio.com/docs/api/2010-04-01/rest/recording)
- [ShortCode](https://www.twilio.com/docs/api/2010-04-01/rest/short-codes)
- [Token](https://www.twilio.com/docs/api/2010-04-01/rest/token)
- [Transcription](https://www.twilio.com/docs/api/2010-04-01/rest/transcription)
- [SipCredentialList](https://www.twilio.com/docs/api/2010-04-01/rest/credential-list)
    - [SipCredential](https://www.twilio.com/docs/api/rest/credential-list#subresources)
- [SipDomain](https://www.twilio.com/docs/api/2010-04-01/rest/domain)
- [SipIPAccessControlList](https://www.twilio.com/docs/api/2010-04-01/rest/ip-access-control-list)
    - [SipIpAddress](https://www.twilio.com/docs/api/rest/ip-access-control-list#subresources)

### Example

```elixir
# Get all the calls in the Call endpoint. Be warned, this will block
# until all the pages of calls have been fetched.
calls = ExTwilio.Call.all

# Create a stream of all the calls
stream = ExTwilio.Call.stream

# Lazily filter calls by duration, then map to get only their SIDs
stream
|> Stream.filter(fn(call) -> call.duration > 120 end)
|> Stream.map(fn(call) -> call.sid end)
|> Enum.into([]) # Only here does any work happen.
# => ["CAc14d7...", "CA649ea861..."]
 
# Get the first page. The meta variable is a map of paging information
# from Twilio.
{:ok, calls, meta} = ExTwilio.Call.list

# Get the next page
{:ok, more_calls, _meta} = ExTwilio.Call.next_page(meta)

# Find a call
{:ok, call} = ExTwilio.Call.find("CA13a9c7f80c6f3761fabae43242b5b6c6")
inspect(call)
# %ExTwilio.Call{
#   account_sid: "ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
#   answered_by: nil, caller_name: "",
#   date_created: "Sat, 14 Mar 2015 14:27:38 +0000",
#   date_updated: "Sat, 14 Mar 2015 14:28:35 +0000", 
#   direction: "outbound-api",
#   duration: "52", 
#   end_time: "Sat, 14 Mar 2015 14:28:35 +0000",
#   forwarded_from: nil, 
#   from: "+1xxxxxxxxxx", 
#   parent_call_sid: nil,
#   phone_number_sid: "", 
#   price: "-0.01500", 
#   price_unit: "USD",
#   sid: "CA13a9c7f80c6f3761fabae43242b5b6c6",
#   start_time: "Sat, 14 Mar 2015 14:27:43 +0000", 
#   status: "completed",
#   to: "+1xxxxxxxxxx",
#   uri: "/2010-04-01/Accounts/ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx/Calls/CAxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.json"
# }

# Update a call
call = ExTwilio.Call.update(call, status: "canceled")

# Get a call's recordings. This pattern is repeated wherever you are 
# getting a nested resource.
recordings = ExTwilio.Recording.all(call: call.sid)

# Destroy a call
ExTwilio.Call.destroy(call)
```

For more in-depth documentation, see the generated docs for each module.

## Contributing

See the [CONTRIBUTING.md](CONTRIBUTING.md) file for contribution guidelines.

## License
ExTwilio is licensed under the MIT license. For more details, see the `LICENSE`
file at the root of the repository. It depends on Elixir, which is under the
Apache 2 license.

Twilio<sup>TM</sup> is trademark of Twilio, Inc.

[hex]: http://hex.pm
