Where to Find the Code
======================
To submit issues and patches please visit https://github.com/DataDog/chef-datadog.
The code is licensed under the Apache License 2.0 (see  LICENSE for details).

[![Chef cookbook](https://img.shields.io/cookbook/v/datadog.svg?style=flat)](https://github.com/DataDog/chef-datadog)
[![Build Status](https://travis-ci.org/DataDog/chef-datadog.svg?branch=master)](https://travis-ci.org/DataDog/chef-datadog)
[![Circle CI](https://circleci.com/gh/DataDog/chef-datadog.svg?style=shield)](https://circleci.com/gh/DataDog/chef-datadog)
[![Coverage Status](https://coveralls.io/repos/DataDog/chef-datadog/badge.svg?branch=master)](https://coveralls.io/r/DataDog/chef-datadog?branch=master)
[![GitHub forks](https://img.shields.io/github/forks/DataDog/chef-datadog.svg)](https://github.com/DataDog/chef-datadog/network)
[![GitHub stars](https://img.shields.io/github/stars/DataDog/chef-datadog.svg)](https://github.com/DataDog/chef-datadog/stargazers)
[![Build Status](https://jenkins-01.eastus.cloudapp.azure.com/job/datadog-cookbook/badge/icon)](https://jenkins-01.eastus.cloudapp.azure.com/job/datadog-cookbook/)

Datadog Cookbook
================

Chef recipes to deploy Datadog's components and configuration automatically.

This cookbook includes new support for Datadog Agent version 6.0, please refer to
the [inline docs](https://github.com/DataDog/chef-datadog/blob/v2.14.0/attributes/default.rb#L31-L68)
for more details on the supported platforms and how to use it.
Log collection is now available with agent 6.0, please refer to the [inline docs](https://github.com/DataDog/chef-datadog/blob/v2.14.0/attributes/default.rb#L371-L376) to enable it.

For general information on the Datadog Agent 6, please refer to the [datadog-agent](https://github.com/DataDog/datadog-agent/) repo.

**NB: This README may refer to features that are not released yet. Please check the README of the
git tag/the gem version you're using for your version's documentation**

Requirements
============
- chef >= 10.14

Platforms
---------

* Amazon Linux
* CentOS
* Debian
* RedHat
* Scientific Linux
* Ubuntu
* Windows (requires chef >= 12.0)

Cookbooks
---------

The following Opscode cookbooks are dependencies:

* `apt`
* `chef_handler`
* `windows`
* `yum`

**Note for Chef 11 users**: please use these additional dependency version contraints for compatibility with Chef 11:

```
cookbook 'apt', '< 4.0'
cookbook 'chef_handler', '< 2.0'
cookbook 'windows', '< 2.0'
cookbook 'yum', '~> 3.0'
```

(in Berkshelf/Librarian format)

Recipes
=======

default
-------
Just a placeholder for now, when we have more shared components they will probably live there.

dd-agent
--------
Installs the Datadog agent on the target system, sets the API key, and start the service to report on the local system metrics

**Notes for Windows**:

* With Chef >= 12.6 _and_ the `windows` cookbook >= 1.39.0, Agent upgrades are known to fail.
  For Chef>=12.6 users on Windows, we recommend pinning the `windows` cookbook to a lower version (`~> 1.38.0` for instance).

  If that's not an option, a known workaround is to use the `remove-dd-agent` recipe (since the `2.5.0` version of the present cookbook) to uninstall the Agent
  prior to any Agent upgrade.

* Because of changes in the Windows Agent packaging and install in version 5.12.0, when upgrading the Agent from versions <= 5.10.1 to versions >= 5.12.0,
  please set the `windows_agent_use_exe` attribute to `true`.

  Once the upgrade is complete, you can leave the attribute to its default value (`false`).

  For more information on these Windows packaging changes, see the related [docs on the dd-agent wiki](https://github.com/DataDog/dd-agent/wiki/Windows-Agent-Installation).

dd-handler
----------
Installs the [chef-handler-datadog](https://rubygems.org/gems/chef-handler-datadog) gem and invokes the handler at the end of a Chef run to report the details back to the newsfeed.

dogstatsd-(python|ruby)
-----------------------
Installs the language-specific libraries to interact with `dogstatsd`.

ddtrace-(python|ruby)
---------------------
Installs the language-specific libraries for application Traces (APM).

other
-----
There are many other integration-specific recipes, that are meant to assist in deploying the correct agent configuration files and dependencies for a given integration.


Usage
=====

### Agent6 Note
Please note the cookbook now supports installing both Agent5 and Agent6 of the datadog agent on debian and RHEL based linux distributions (not yet supported on windows). By default versions `<=2.x` of the cookbook will default to install Agent5, you may however override this behavior with the `node['datadog']['agent6']` attribute.
  ```
  default_attributes(
    'datadog' => {
      'agent6' => true
    }
  )
  ```

Additional attributes are available to have finer control over how you install agent6. These are agent6 counterparts to several well known agent5 attributes (code [here](https://github.com/DataDog/chef-datadog/blob/master/attributes/default.rb#L31-L69)):
 * `agent6_version`: should allow you to pin the agent version.
 * `agent6_package_action`: defaults to `'install'`, may be set to `'upgrade'` to always upgrade to latest.
 * `agent6_aptrepo`: desired APT repo for the agent. Defaults to `http://apt.datadoghq.com`
 * `agent6_aptrepo_dist`: desired distribution for the APT repo. Defaults to `stable`
 * `agent6_yumrepo`: desired YUM repo for the agent. Defaults to `https://yum.datadoghq.com/stable/6/x86_64/`

Should wish to add additional elements to the agent6 configuration file, typically `/etc/datadog/datadog.yaml` you may use the `node['datadog']['extra_config']` attribute. This attribute is a hash and will be marshaled into the configuration file accordingly. For instance, to enable the `apm` tracing agent bundled with agent6:
  ```
  default_attributes(
    'datadog' => {
      'extra_config' => {
        'apm_config' => {
          'enabled' => true
        }
      }
    }
  )
  ```

or to enable both process agent and apm:
  ```
  default_attributes(
    'datadog' => {
      'extra_config' => {
        'apm_config' => {
          'enabled' => true
        },
        'process_config' => {
          'enabled' => 'disabled'
        }
      }
    }
  )
  ```
  NB: please take a look at the process and apm agents for more details regarding configuration options. [APM Trace Agent](https://github.com/DataDog/datadog-trace-agent) and [Process Agent](https://github.com/DataDog/datadog-process-agent)


### Instructions

1. Add this cookbook to your Chef Server, either by installing with knife or by adding it to your Berksfile:
  ```
  cookbook 'datadog', '~> 2.14.0'
  ```
2. Add your API Key either:
  * as a node attribute via an `environment` or `role`, or
  * as a node attribute by declaring it in another cookbook at a higher precedence level, or
  * in the node `run_state` by setting `node.run_state['datadog']['api_key']` in another cookbook preceding `datadog`'s recipes in the run_list. This approach has the benefit of not storing the credential in clear text on the Chef Server.
3. Create an 'application key' for `chef_handler` [here](https://app.datadoghq.com/account/settings#api), and add it as a node attribute or in the run state, as in Step #2.

   NB: if you're using the run state to store the api and app keys you need to set them at compile time before `datadog::dd-handler` in the run list.

4. Associate the recipes with the desired `roles`, i.e. "role:chef-client" should contain "datadog::dd-handler" and a "role:base" should start the agent with "datadog::dd-agent".  Here's an example role with both recipes:
  ```
  name 'example'
  description 'Example role using DataDog'

  default_attributes(
    'datadog' => {
      'api_key' => 'api_key',
      'application_key' => 'app_key'
    }
  )

  run_list %w(
    recipe[datadog::dd-agent]
    recipe[datadog::dd-handler]
  )
  ```
  NB: remember to set `agent6` attribute in the `datadog` hash if you'd like to install agent6.

5. Wait until `chef-client` runs on the target node (or trigger chef-client manually if you're impatient)

We are not making use of data_bags in this recipe at this time, as it is unlikely that you will have more than one API key and one application key.

For more deployment details, visit the [Datadog Documentation site](http://docs.datadoghq.com/).

AWS OpsWorks Chef Deployment
----------------------------

1. Add Chef Custom JSON:
  ```json
  {"datadog":{"api_key": "<API_KEY>", "application_key": "<APP_KEY>"}}
  ```

2. Include the recipe in install-lifecycle recipe:
  ```ruby
  include_recipe 'datadog::dd-agent'
  ```
