# Ardana automated testing

Starting with SOC8, Ardana can be deployed and tested using sources coming from the
[internal build service](https://build.suse.de/project/show/Devel:Cloud:8:Staging),
automated through jobs running in the [ci.nue.suse.com Jenkins service](https://ci.nue.suse.com/job/openstack-ardana)
and consuming the virtualized resources in [the engineering cloud](https://engcloud.prv.suse.net/project/stacks/).
This process is implemented by the [Jenkins pipeline jobs](https://github.com/SUSE-Cloud/automation/blob/master/jenkins/ci.suse.de/pipelines)
and the [ansible playbooks](https://github.com/SUSE-Cloud/automation/blob/master/scripts/jenkins/ardana/ansible) located
in this repository.

The easiest way to experiment with an automated Ardana deployment is by [triggering a customized Ardana Jenkins job build](#deploying-and-testing-ardana-through-jenkins).
Alternatively, the same ansible playbooks called by the Ardana Jenkins jobs can be used to deploy and test Ardana independently
from Jenkins, by [setting up a local test environment](#deploying-and-testing-ardana-manually) on any host that
has access to the engineering cloud and by running the playbooks there.

## Deploying and testing Ardana through Jenkins

The [Jenkins pipeline job called `openstack-ardana`](https://ci.nue.suse.com/job/openstack-ardana/) (defined in the
[`automation` git repo](https://github.com/SUSE-Cloud/automation/blob/master/jenkins/ci.suse.de/openstack-ardana.yaml))
can be used to create a new virtual environment in the [engineering cloud](https://engcloud.prv.suse.net/) or to deploy
a bare-metal environment on one of the QA hardware clusters. The deployment can then be accessed via the IP address
allocated to the deployer node and used for debugging.

### Prerequisites

To be able to use the Jenkins job and login to a deployed environment, the following are needed:

* the cloud team login credentials for https://ci.nue.suse.com (ask in the cloud-team RocketChat channel if you don't
know what they are, and someone will PM you)
* your public ssh key added to [the list of keys](https://github.com/SUSE-Cloud/automation/blob/master/scripts/jenkins/ardana/ansible/group_vars/all/ssh_pub_keys.yml)(via normal github pull requests)

### Deploying a Jenkins Ardana environment

An Ardana deployment can be started via [Build with
parameters](https://ci.nue.suse.com/job/openstack-ardana/build). The
available parameters should be self-descriptive, but the [Customized deployments](#customized-deployments) section
will provide more information on the provided options. Here are some best
practices:

* For a virtual cloud deployment, use your Rocket/irc nickname as ```ardana_env```. If you need to deploy several Ardana setups,
  use a unique ```ardana_env``` value for each one. Note that ```ardana_env``` values starting with
  `qe` (e.g `qe101`, `qe102`) are reserved for QA baremetal deployments.
* Select a ```cleanup``` value to decide what happens with the Ardana virtual cloud deployment after the job completes
* To test a different input model other than the default `std-min`, adjust the
  ```model``` parameter in the deployment
* To test a different software media, use another `cloudsource` value. The available values and their meaning are documented
[here](../../scripts/jenkins/ardana/ansible/roles/setup_zypper_repos/README.md)
* To test a custom automation repository, push to your fork and adjust the
  ```git_automation_repo``` and ```git_automation_branch``` parameter values

The IP address allocated to the deployer node will be available in several places:
* the Jenkins job build name will be updated automatically to include it, when it becomes available
* it will be printed in the Jenkins job log in several places, e.g.:

```
10:49:22 [Prepare virtual cloud] ******************************************************************************
10:49:22 [Prepare virtual cloud] ** The deployer for the 'cloud-ardana-ci-slot1' virtual environment is reachable at:
10:49:22 [Prepare virtual cloud] **
10:49:22 [Prepare virtual cloud] **        ssh root@10.86.1.146
10:49:22 [Prepare virtual cloud] **
10:49:22 [Prepare virtual cloud] ******************************************************************************
```

* it will also be available as output [in the Heat stack](https://engcloud.prv.suse.net/project/stacks) as the
variable ```admin-floating-ip```, for virtual cloud deployments

The environment can then accessed by logging in as `root` or `ardana` and using the floating IP.


**PLEASE DELETE YOUR ENVIRONMENT IN THE ENGINEERING CLOUD AFTER YOU ARE DONE** otherwise we'll run into quota
limits in our project. Instructions on how to clean up the environment will be printed in the Jenkins job log
at the end of the run, e.g.:

```
13:34:20 ******************************************************************************
13:34:20 ** The deployer for the 'alan-turing' virtual environment is reachable at:
13:34:20 **
13:34:20 **        ssh root@10.86.1.235
13:34:20 **
13:34:20 ** Please delete the 'openstack-ardana-alan-turing' stack when you're done,
13:34:20 ** by using one of the following methods:
13:34:20 **
13:34:20 **  1. log into the ECP at https://engcloud.prv.suse.net/project/stacks/
13:34:20 **  and delete the stack manually, or
13:34:20 **
13:34:20 **  2. (preferred) trigger a manual build for the openstack-ardana-heat job at
13:34:20 **  https://ci.nue.suse.com/job/openstack-ardana-heat/build and use the
13:34:20 **  same 'alan-turing' ardana_env value and the 'delete' action for the
13:34:20 **  parameters
13:34:20 **
13:34:20 ******************************************************************************
```


### Testing a predeployed Jenkins Ardana environment

Aside from manual testing, any of the following pipeline Jenkins jobs can be manually triggered via `Build with Parameters`
to run on an Ardana environment that was previously deployed using the Ardana Jenkins jobs. The target Ardana virtual or
bare-metal environment must be indicated via the `ardana_env` parameter value:

* [openstack-ardana-qa-tests](https://ci.nue.suse.com/job/openstack-ardana-tests/build) : runs tempest or QA test cases
* [openstack-ardana-update](https://ci.nue.suse.com/job/openstack-ardana-update/build) : automates the maintenance update
workflow to install package updates on the Ardana environment
* [openstack-ardana-update-and-test](https://ci.nue.suse.com/job/openstack-ardana-update-and-test/build) : combines the two
jobs above into one pipeline


### Customized deployments

TBD:

custom cloudsource
different input model
gerrit changes
extra repositories
MUs
tempest run filter
QA test cases
  - TODO: link to adding new test cases
automation repo PR
generated input model
  - different scenario (standard, split, mid-size)
  - control number of nodes allocated to each cluster
  - integrated/standalone deployer
  - TODO: disabled services
  - TODO: link to adding new scenarios
QA physical deployment
custom automation repo fork/branch


## Deploying and testing Ardana manually

The ansible playbooks used by the Jenkins job can alternatively be executed manually from any host,
without involving Jenkins at all, to deploy and test either a virtual Ardana environment hosted in the engineering
cloud, or an Ardana QA bare-metal environment.

### Prerequisites

To be able to deploy an Ardana environment using the Ardana automation ansible playbooks, the following are needed:

* an [engineering cloud](https://engcloud.prv.suse.net) account (for virtual Ardana environments)
* your public ssh key added to [the list of keys](https://github.com/SUSE-Cloud/automation/blob/master/scripts/jenkins/ardana/ansible/group_vars/all/ssh_pub_keys.yml)(via normal github pull requests)
* a host with access to the [engineering cloud](https://engcloud.prv.suse.net) (for virtual Ardana environments)
* a host with access to the QA hardware clusters (for QA bare-metal Ardana environments)

### Test environment setup

Setting up a local Ardana test environment:

* clone the [automation repository](https://github.com/SUSE-Cloud/automation) locally
* for building test packages from gerrit changes, the osc utility needs to be correctly installed and configured on
the local host.
* set up an OpenStack cloud configuration reflecting your engineering cloud credentials (for virtual Ardana environments).
To do that, create an `~/.config/openstack/clouds.yaml` file with the following contents reflecting your engineering cloud account:

```
clouds:
  engcloud-cloud-ci:
    region_name: CustomRegion
    auth:
      auth_url: https://engcloud.prv.suse.net:5000/v3
      username: <your ldap user name>
      password: <your ldap password>
      project_name: cloud
      project_domain_name: default
      user_domain_name: ldap_users
    identity_api_version: 3
    cacert: /usr/share/pki/trust/anchors/SUSE_Trust_Root.crt.pem
```

To verify that the test environment is properly set up:

* run an openstack CLI command, e.g.:

```
openstack --os-cloud engcloud-cloud-ci stack list
```

### Manual Ardana deployment

A bash script is provided to help deploying Ardana manually. The script sets up a virtual environment with ansible and all
requirements and mimics the steps of the openstack-ardana job by calling the ansible playbooks in the appropriate
sequence with the appropriate parameters, which are taken from `input.yml`.

Before running the script you need to configure the parameters in the `input.yml` file to fit how you want ardana to be
deployed. The options available are basically a subset of the inputs from the openstack-ardana jenkins job.

Deploying Ardana:

* Go to `scripts/jenkins/ardana/manual` directory
* Edit the `input.yml` file
* Run `deploy-ardana.sh`

```
cd scripts/jenkins/ardana/manual
vim input.yml
./deploy-ardana.sh
```

Alternatively you can also call each step individually (after configuring `input.yml`) by sourcing the script library.
E.g.:

```
cd scripts/jenkins/ardana/manual
source lib.sh
setup_ansible_venv
mitogen_enable
prepare_input_model
prepare_infra
...
```

## Custom Ardana Gerrit CI builds

The default integration test job launched by the Gerrit Jenkins CI uses
a `standard` generated input model, with a standalone deployer, 2
controller nodes and one SLES compute node. It also runs against a
`develcloud` cloud media, on top of which it applies the most recent
Ardana ansible changes and, of course, the target Gerrit change under
test and all its Gerrit change dependencies.

This is sufficient for most cases, but sometimes there are special
circumstances that require one or more additional, customized
`openstack-ardana` type integration jobs to be executed against the same
Gerrit change and to have them report their results back to Gerrit,
where they can be tracked and verified by all stake holders.

This is now supported by allowing users to trigger manual builds for
the parameterized Jenkins job that runs the Gerrit CI for either
[cloud 8](https://ci.nue.suse.com/job/openstack-ardana-gerrit-cloud8/build)
or [cloud 9](https://ci.nue.suse.com/job/openstack-ardana-gerrit-cloud9/build),
and supplying custom values for its parameters, to achieve different
results:

* `GERRIT_CHANGE_NUMBER` - this parameter must be set to the target Gerrit
change number value

* `voting` - this flag controls whether the Gerrit job posts `Verify+2`
or `Verify-2` label values on the Gerrit change. Non-voting jobs also
post progress and outcome messages, but do not alter the `Verify` label
value. There can be at most one voting job running for a Gerrit change
at any given time. The most recent voting job automatically cancels
older voting jobs running for the same Gerrit change.

* `gerrit_context` - this is a string value that uniquely identifies
a job build within the set of builds running against the same Gerrit
change. A unique `gerrit_context` parameter value should be used for
each additional job that is manually triggered against a Gerrit
change, otherwise it will supersede and automatically cancel the
previous jobs running with the same `gerrit_context` value on the same
Gerrit change.
This parameter should be set to a short string describing what the job
actually achieves, as it is included both in the Gerrit build status
reports and the Jenkins build description.

* `integration_test_job` - the name of an existing Ardana Jenkins job
that is triggered to run integration tests on the target Gerrit change.
This parameter may be used to override the default
`cloud-ardana8-job-std-min-gerrit-x86_64` and `cloud-ardana8-job-std-min-gerrit-x86_64`
jobs currently employed by the Ardana CI to run integration tests.
If the custom integration job also requires its parameters to be overridden,
this can be achieved via the `extra_params` parameter.

* `git_automation_repo` and `git_automation_branch` - can be used to
point Jenkins to a different automation repository branch or fork.

* `extra_repos` - allows the user to configure custom zypper repositories
in addition to those implied by the target `cloudsource` value.

* `extra_params` - this parameter is present for most Ardana Jenkins jobs.
It can be used to inject additional parameters into the build, which are
reflected in several places:
  * the list of environment variables accessible to shell scripts
  * the list of ansible extra variables, which can override all other
  variables used by playbooks
  * the `extra_params` parameter values of jobs triggered downstream,
  which means the effect is cascaded down to the entire job build
  hierarchy


Following are a few examples of use-cases where this feature comes in
handy.

### Custom integration jobs

A Gerrit change may need to be tested against a different input model
scenario or cloud configuration. Here are some practical examples of this
situation:

  * a Gerrit change that impacts the RHEL compute node support _also_ needs
  to be tested against a cloud input model that includes RHEL (CentOS)
  compute nodes.
  E.g. to have this running for a cloud9 Gerrit change, trigger a
  [manual cloud 9 Gerrit CI build](https://ci.nue.suse.com/job/openstack-ardana-gerrit-cloud9/build),
  with the following parameter values:

    * `voting` unset, to have the new build run in addition to the official
    voting CI job, as a non-voting job
    * `integration_test_job` set to point at a job running an input model
    with RHEL compute nodes (for example
    [cloud-ardana9-job-std-min-centos-gerrit-x86_64](https://ci.nue.suse.com/job/cloud-ardana9-job-std-min-centos-gerrit-x86_64))
    * a custom `gerrit_context` value to indicate what the new job does
    (e.g. `std-min-centos`).

  * a Gerrit change that touches on functionality that can only be tested
  on bare-metal deployments (e.g. cobbler) needs to be tested against
  a QA bare-metal environment _instead of_ the virtual one.
  E.g. to have this running for a cloud8 Gerrit change, trigger a
  [manual cloud 8 Gerrit CI build](https://ci.nue.suse.com/job/openstack-ardana-gerrit-cloud8/build)
  with the following parameter values:

    * `ardana_env` set to point to one of the available QA bare-metal
    environments (contact the QA team to find one that is available)
    * `reserve_env` unchecked (unless the `ardana_env` QA environment also
    has an associated [Lockable Resource](https://ci.nue.suse.com/lockable-resources) )
    * `voting` set, to have the new build replace the official voting CI
    job and determine whether the Gerrit change can be merged or not
    * `integration_test_job` set to point at a job running an input model
    that can actually be deployed on bare-metal (for example
  [cloud-ardana8-job-entry-scale-kvm-gerrit-x86_64](https://ci.nue.suse.com/job/cloud-ardana8-job-entry-scale-kvm-gerrit-x86_64))
    * a custom `gerrit_context` value to indicate what the new job does
    (e.g. `entry-scale-bare-metal`).

As can be seen, these use-cases are supported by allowing the user to
specify a different Ardana integration Jenkins job to be run against
the Gerrit change instead of the default one.
The custom `integration_test_job` Jenkins job can either be one of the
existing jobs that have already been provided for this purpose - those
with their name ending in `-gerrit-x86_64` (**recommended**), or it can even
be the fully customizable core [openstack-ardana](https://ci.nue.suse.com/job/openstack-ardana)
job itself, in which case some or all of its parameters may need to be
customized by using the `extra_params` parameter.

E.g. to have a fully customizable `openstack-ardana` job running for a cloud9 Gerrit
change, one might trigger a [manual cloud 9 Gerrit CI build](https://ci.nue.suse.com/job/openstack-ardana-gerrit-cloud9/build)
with the following parameter values:

  * `ardana_env` set to the IRC or LDAP username of the user (i.e. this
  will be a private Ardana ECP deployment environment that remains running
  and can be accessed after the job is done)
  * `reserve_env` unchecked
  * `voting` set, to have the new build replace the official voting CI
  job and determine whether the Gerrit change can be merged or not
  * `integration_test_job` set to `openstack-ardana`
  * a custom `gerrit_context` value to indicate what the new job does
  (e.g. `my-very-special-input-model`)
  * the multi-line `extra_params` parameter set to the list of
  [openstack-ardana](https://ci.nue.suse.com/job/openstack-ardana/build)
  build parameters that are mandatory (need to be supplied) or need to be
  overridden (different than their default values). For example, the following
  set of `extra_params` parameters can be used to deploy a `demo` like input model
  (integrated deployer, one controller, one SLES compute, monasca service disabled):

  ```
  scenario_name=standard
  clm_model=integrated
  controllers=1
  sles_computes=1
  disabled_services=monasca|logging|ceilometer|cassandra|kafka|spark|storm|octavia
  ses_rgw_enabled=false
  tempest_filter_list=ci
  ```


If a particular customized `openstack-ardana` scenario needs to be used
often, it is recommended that it be added with a GitHub pull-request to
the list of `-gerrit-x86_64` jobs predefined for
[cloud8](https://github.com/SUSE-Cloud/automation/blob/master/jenkins/ci.suse.de/cloud-ardana8-gerrit.yaml)
and/or [cloud9](https://github.com/SUSE-Cloud/automation/blob/master/jenkins/ci.suse.de/cloud-ardana9-gerrit.yaml),
and thus enable other users to point to it by name.

### Custom package repositories

The user may need additional zypper repositories to be configured, for
example:

* if there is a two-way dependency between a Gerrit change on one hand,
and one or more OBS/IBS package changes on the other hand, meaning that
they must be tested together to ensure that neither breaks the CI, before
they can be merged together into the main code and package streams
* when the Gerrit change implements a new piece of functionality that
goes hand-in-hand with a new set of packages or package updates that are
not yet available in the development cloud build (e.g. they reside in an
OBS home project or another non-official project while still being
developed and tested).

The recommended approach for this use-case is that test packages be
created in an OBS or IBS project with package publishing enabled, then
the URL to the `.repo` file generated in the repository can be supplied
as the `extra_repos` parameter value when triggering custom manual
Gerrit CI Jenkins builds.

**IMPORTANT**: currently, this feature is only supported for packages that need
to be installed on the deployer node - Ardana ansible and venv packages
(see [SCRD-7800](https://jira.suse.de/browse/SCRD-7800)). If the packages
in the extra repositories also need to be installed on the non-deployer
nodes, it will not work until SCRD-7800 is solved.

### Custom automation scripts or jobs

While new CI test coverage is still being developed in a separate
automation repository fork or branch (or pull-request), that doesn't mean
it cannot yet be executed to validate open Gerrit changes.

This is useful, for example, when someone has a set of open Gerrit changes
that implement a new piece of functionality or use-case and someone else
(or the same person) works in parallel on providing automation scripts
that target that same functionality or use-case in the CI.

When triggering manual Gerrit CI builds, an alternative git source can
be used via the `git_automation_repo` and `git_automation_branch`
parameters. There are some limitations to this feature though:

* if the alternative git branch/fork also creates new JJB Jenkins jobs,
these will not be available in Jenkins yet until changes are merged in the
master branch or until created manually by the user
* if the alternative git branch/fork updates the configuration of
existing JJB Jenkins jobs (e.g. adds, removes or replaces parameters),
these changes will also not be available in Jenkins. In this case, the
recommendation is to use the `extra_params` feature, if possible, to
achieve the same effect as would updating the existing Jenkins jobs,
or create temporary new jobs that are copies of the existing ones that
are updated in the automation git branch/fork

## Ardana CI jobs

The majority of Ardana SUSE CI Jenkins jobs are [Jenkins pipeline jobs](https://jenkins.io/doc/book/pipeline/). These offer some
advantages over the classical job types, which are heavily exploited by the Ardana Jenkins jobs:

* better usability: a job run can be broken in stages, their pass/fail status and logs individually visualized,
especially useful in the Blue Ocean Jenkins UI
* more flexible
* TBD

###


TBD: describe the existing jobs, their purpose, triggers,
link to the detailed pipeline description for every job.

### Prepare BM cloud

### Prepare virtual cloud

### Build test packages

### Bootstrap CLM

### Bootstrap nodes

### Deploy cloud

### Tempest

### Deploy CaaSP

## Ardana Jenkins pipelines

### Integration pipeline

Describe the pipeline job, stages, link to detail docs for various features/roles.,
Diagram with job stages/hierarchy.

### Gerrit pipeline

Prepare BM cloud


## Automated checks

Some extra checks are automatically run by the Jenkins job:

- [rpm file checks](rpm-file-checks.md)
