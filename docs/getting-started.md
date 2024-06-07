<!--
  ~ Copyright (c) 2023-2024 Arista Networks, Inc.
  ~ Use of this source code is governed by the Apache License 2.0
  ~ that can be found in the LICENSE file.
  -->

# Getting Started

This section shows how to use ANTA with basic configuration. All examples are based on Arista Test Drive (ATD) topology you can access by reaching out to your preferred SE.

## Installation

The easiest way to install ANTA package is to run Python (`>=3.9`) and its pip package to install:

```bash
pip install anta[cli]
```

For more details about how to install package, please see the [requirements and installation](./requirements-and-installation.md) section.

## Configure Arista EOS devices

For ANTA to be able to connect to your target devices, you need to configure your management interface

```eos
vrf instance MGMT
!
interface Management0
   description oob_management
   vrf MGMT
   ip address 192.168.0.10/24
!
```

Then, configure access to eAPI:

```eos
!
management api http-commands
   protocol https port 443
   no shutdown
   vrf MGMT
      no shutdown
   !
!
```

## Create your inventory

ANTA uses an inventory to list the target devices for the tests. You can create a file manually with this format:

```yaml
anta_inventory:
  hosts:
  - host: 192.168.0.10
    name: spine01
    tags: ['fabric', 'spine']
  - host: 192.168.0.11
    name: spine02
    tags: ['fabric', 'spine']
  - host: 192.168.0.12
    name: leaf01
    tags: ['fabric', 'leaf']
  - host: 192.168.0.13
    name: leaf02
    tags: ['fabric', 'leaf']
  - host: 192.168.0.14
    name: leaf03
    tags: ['fabric', 'leaf']
  - host: 192.168.0.15
    name: leaf04
    tags: ['fabric', 'leaf']
```

> You can read more details about how to build your inventory [here](usage-inventory-catalog.md#create-an-inventory-file)

## Test Catalog

To test your network, ANTA relies on a test catalog to list all the tests to run against your inventory. A test catalog references python functions into a yaml file.

The structure to follow is like:

```yaml
<anta_tests_submodule>:
  - <anta_tests_submodule function name>:
      <test function option>:
        <test function option value>
```

> You can read more details about how to build your catalog [here](usage-inventory-catalog.md#test-catalog)

Here is an example for basic tests:

```yaml
# Load anta.tests.software
anta.tests.software:
  - VerifyEOSVersion: # Verifies the device is running one of the allowed EOS version.
      versions: # List of allowed EOS versions.
        - 4.25.4M
        - 4.26.1F
        - '4.28.3M-28837868.4283M (engineering build)'
  - VerifyTerminAttrVersion:
      versions:
        - v1.22.1

anta.tests.system:
  - VerifyUptime: # Verifies the device uptime is higher than a value.
      minimum: 1
  - VerifyNTP:
  - VerifySyslog:

anta.tests.mlag:
  - VerifyMlagStatus:
  - VerifyMlagInterfaces:
  - VerifyMlagConfigSanity:

anta.tests.configuration:
  - VerifyZeroTouch: # Verifies ZeroTouch is disabled.
  - VerifyRunningConfigDiffs:
```

## Test your network

### Basic usage in a python script

```python
--8<-- "anta_runner.py"
```

### CLI

ANTA comes with a generic CLI entrypoint to run tests in your network. It requires an inventory file as well as a test catalog.

This entrypoint has multiple options to manage test coverage and reporting.

```bash
--8<-- "anta_help.txt"
```

```bash
--8<-- "anta_nrfu_help.txt"
```

To run the NRFU, you need to select an output format amongst ["json", "table", "text", "tpl-report"]. For a first usage, `table` is recommended.  By default all test results for all devices are rendered but it can be changed to a report per test case or per host

#### Default report using table

```bash
anta nrfu \
    --username tom \
    --password arista123 \
    --enable \
    --enable-password t \
    --inventory .personal/inventory_atd.yml \
    --catalog .personal/tests-bases.yml \
    table --tags leaf


╭────────────────────── Settings ──────────────────────╮
│ Running ANTA tests:                                  │
│ - ANTA Inventory contains 6 devices (AsyncEOSDevice) │
│ - Tests catalog contains 10 tests                    │
╰──────────────────────────────────────────────────────╯
[10:17:24] INFO     Running ANTA tests...                                                                                                           runner.py:75
  • Running NRFU Tests...100% ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 40/40 • 0:00:02 • 0:00:00

                                                                       All tests results
┏━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━┓
┃ Device IP ┃ Test Name                ┃ Test Status ┃ Message(s)       ┃ Test description                                                     ┃ Test category ┃
┡━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━┩
│ leaf01    │ VerifyEOSVersion         │ success     │                  │ Verifies the device is running one of the allowed EOS version.       │ software      │
│ leaf01    │ VerifyTerminAttrVersion  │ success     │                  │ Verifies the device is running one of the allowed TerminAttr         │ software      │
│           │                          │             │                  │ version.                                                             │               │
│ leaf01    │ VerifyUptime             │ success     │                  │ Verifies the device uptime is higher than a value.                   │ system        │
│ leaf01    │ VerifyNTP                │ success     │                  │ Verifies NTP is synchronised.                                        │ system        │
│ leaf01    │ VerifySyslog             │ success     │                  │ Verifies the device had no syslog message with a severity of warning │ system        │
│           │                          │             │                  │ (or a more severe message) during the last 7 days.                   │               │
│ leaf01    │ VerifyMlagStatus         │ skipped     │ MLAG is disabled │ This test verifies the health status of the MLAG configuration.      │ mlag          │
│ leaf01    │ VerifyMlagInterfaces     │ skipped     │ MLAG is disabled │ This test verifies there are no inactive or active-partial MLAG      │ mlag          │
[...]
│ leaf04    │ VerifyMlagConfigSanity   │ skipped     │ MLAG is disabled │ This test verifies there are no MLAG config-sanity inconsistencies.  │ mlag          │
│ leaf04    │ VerifyZeroTouch          │ success     │                  │ Verifies ZeroTouch is disabled.                                      │ configuration │
│ leaf04    │ VerifyRunningConfigDiffs │ success     │                  │                                                                      │ configuration │
└───────────┴──────────────────────────┴─────────────┴──────────────────┴──────────────────────────────────────────────────────────────────────┴───────────────┘
```

#### Report in text mode

```bash
$ anta nrfu \
    --username tom \
    --password arista123 \
    --enable \
    --enable-password t \
    --inventory .personal/inventory_atd.yml \
    --catalog .personal/tests-bases.yml \
    text --tags leaf

╭────────────────────── Settings ──────────────────────╮
│ Running ANTA tests:                                  │
│ - ANTA Inventory contains 6 devices (AsyncEOSDevice) │
│ - Tests catalog contains 10 tests                    │
╰──────────────────────────────────────────────────────╯
[10:20:47] INFO     Running ANTA tests...                                                                                                           runner.py:75
  • Running NRFU Tests...100% ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 40/40 • 0:00:01 • 0:00:00
leaf01 :: VerifyEOSVersion :: SUCCESS
leaf01 :: VerifyTerminAttrVersion :: SUCCESS
leaf01 :: VerifyUptime :: SUCCESS
leaf01 :: VerifyNTP :: SUCCESS
leaf01 :: VerifySyslog :: SUCCESS
leaf01 :: VerifyMlagStatus :: SKIPPED (MLAG is disabled)
leaf01 :: VerifyMlagInterfaces :: SKIPPED (MLAG is disabled)
leaf01 :: VerifyMlagConfigSanity :: SKIPPED (MLAG is disabled)
[...]
```

#### Report in JSON format

```bash
$ anta nrfu \
    --username tom \
    --password arista123 \
    --enable \
    --enable-password t \
    --inventory .personal/inventory_atd.yml \
    --catalog .personal/tests-bases.yml \
    json --tags leaf

╭────────────────────── Settings ──────────────────────╮
│ Running ANTA tests:                                  │
│ - ANTA Inventory contains 6 devices (AsyncEOSDevice) │
│ - Tests catalog contains 10 tests                    │
╰──────────────────────────────────────────────────────╯
[10:21:51] INFO     Running ANTA tests...                                                                                                           runner.py:75
  • Running NRFU Tests...100% ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 40/40 • 0:00:02 • 0:00:00
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ JSON results of all tests                                                                                                                                    │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
[
  {
    "name": "leaf01",
    "test": "VerifyEOSVersion",
    "categories": [
      "software"
    ],
    "description": "Verifies the device is running one of the allowed EOS version.",
    "result": "success",
    "messages": [],
    "custom_field": "None",
  },
  {
    "name": "leaf01",
    "test": "VerifyTerminAttrVersion",
    "categories": [
      "software"
    ],
    "description": "Verifies the device is running one of the allowed TerminAttr version.",
    "result": "success",
    "messages": [],
    "custom_field": "None",
  },
[...]
]
```

You can find more information under the __usage__ section of the website
