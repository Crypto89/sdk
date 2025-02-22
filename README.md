# Avi SDK and Utilities

[![Build Status](https://travis-ci.org/avinetworks/sdk.svg?branch=17.1.4_pytest)](https://travis-ci.org/avinetworks/sdk)

This repository includes API documentation, SDK and sample source to integrate
into the Avi Solution. Below is brief description of the contents.

- **doc**: Avi API documentation includes documentation for the load balancer
objects, Analytics Metrics, and Log Analytics APIs.
- **python**: Source for the Python SDK. The pip package can be downloaded from
[Releases](https://github.com/avinetworks/sdk/releases "Avi SDK Releases").
Here are list of important SDK directories:
    - **samples**: Python samples are in the python/avi/sdk/samples directory.
        - **autoscale**: Gives ControlScript examples for
        server autoscaling.
        - **heat**: Provides a heat example for pool servers that can be used
        with the server autoscale feature and ControlScripts.
        - **virtualservice_examples_api**: Provides examples of programmatically
        creating most common virtual services, such as basic VS, SSL VS, analytics
        APIs, tenant-based APIs, etc.
    - **utils**: Useful utilities for devops automation.
        - **f5_converter**: Is a utility for converting an F5 configuration into
        an Avi Vantage configuration.
        - **httppolicyset_templates**: Provides easy-to-use templates for
        creating HTTP request and redirect policies for most common use cases.
        - **Mesos**: Provides CRUD APIs to create a Marathon app with Avi labels.

# Installation
Pip packages are hosted on GitHub. They can be installed simply as:
### Avi SDK Install
```sh
$ pip install avisdk
```
### Avi Migration Tools Install - installs F5-to-Avi and NetScaler-to-Avi conversion tools
```sh
$ pip install avimigrationtools
```

### Python Virtual-Environment-based Installation
It is recommended to use a virtual-environment-based installation if you are just
experimenting with the SDK or F5 converter.

```sh
$ apt-get install python-dev python-pip python-virtualenv python-cffi libssl-dev libffi-dev
$ virtualenv avi
$ cd avi
$ source bin/activate
$ pip install avimigrationtools avisdk
$ f5_converter.py -h
$ netscaler_converter.py -h
$ deactivate
```

# Usage
## ApiSession Usage

```python
from avi.sdk.avi_api import ApiSession
# create Avi API Session
api = ApiSession.get_session("10.10.10.42", "admin", "something", tenant="admin")

# create virtualservice using pool sample_pool
pool_obj = api.get_object_by_name('pool', 'sample_pool')
pool_ref = api.get_obj_ref(pool_obj)
services_obj = [{'port': 80, 'enable_ssl': False}]
vs_obj = {'name': 'sample_vs', 'ip_address': {'addr': '11.11.11.42', 'type': 'V4'},
         'services': services_obj, 'pool_ref': pool_ref}
resp = api.post('virtualservice', data=vs_obj)

# print list of all virtualservices
resp = api.get('virtualservice')
for vs in resp.json()['results']:
    print vs['name']

# delete virtualservice
resp = api.delete_by_name('virtualservice', 'sample_vs')
```

If ApiSession is invoked in the context of a control script, then token can be used for authentication.
Along with that, information regarding username and tenant information can also be retrieved as follows:
```python
token=os.environ.get('API_TOKEN')
user=os.environ.get('USER')
tenant=os.environ.get('TENANT')
api = ApiSession.get_session("localhost", user, token=token, tenant=tenant)
```
# SAML Authentication Usage
### prerequisite:
1. SAML configured/enabled Controller.

To set up SAML SSO controller, please refer the below link. 
(https://avinetworks.com/docs/17.2/single-sign-on-with-saml/)

Currently, the SDK supports two IDPs for SAML-based authentication:
1) Okta
2) Onelogin

## SAML-based Session Usage for the Okta IDP

```python
from avi.sdk.saml_avi_api import OktaSAMLApiSession
# create Avi API Session
api = OktaSAMLApiSession("10.10.10.42", "okta_username", "okta_password")

# create virtualservice using pool sample_pool
pool_obj = api.get_object_by_name('pool', 'sample_pool')
pool_ref = api.get_obj_ref(pool_obj)
services_obj = [{'port': 80, 'enable_ssl': False}]
vs_obj = {'name': 'sample_vs', 'ip_address': {'addr': '11.11.11.42', 'type': 'V4'},
         'services': services_obj, 'pool_ref': pool_ref}
resp = api.post('virtualservice', data=vs_obj)

# print list of all virtualservices
resp = api.get('virtualservice')
for vs in resp.json()['results']:
    print vs['name']

# delete virtualservice
resp = api.delete_by_name('virtualservice', 'sample_vs')
```

## SAML-based Session Usage for the OneLogin IDP

```python
from avi.sdk.saml_avi_api import OneloginSAMLApiSession
# create Avi API Session
api = OneloginSAMLApiSession("10.10.10.42", "onelogin_username", "onelogin_password")

# create virtualservice using pool sample_pool
pool_obj = api.get_object_by_name('pool', 'sample_pool')
pool_ref = api.get_obj_ref(pool_obj)
services_obj = [{'port': 80, 'enable_ssl': False}]
vs_obj = {'name': 'sample_vs', 'ip_address': {'addr': '11.11.11.42', 'type': 'V4'},
         'services': services_obj, 'pool_ref': pool_ref}
resp = api.post('virtualservice', data=vs_obj)

# print list of all virtualservices
resp = api.get('virtualservice')
for vs in resp.json()['results']:
    print vs['name']

# delete virtualservice
resp = api.delete_by_name('virtualservice', 'sample_vs')
```

SAML session can also be invoked by following:
```
api = ApiSession.get_session("10.10.10.42", "onelogin_username", "onelogin_password", idp_class=OneloginSAMLApiSession)
```
```
api = ApiSession.get_session("10.10.10.42", "onelogin_username", "onelogin_password", idp_class=OktaSAMLApiSession)
```
#### F5 Converter Usage
See all the F5 converter options
```sh
f5_converter.py -h
```
Convert bigip.conf into Avi configuration. Output is in output directory
```sh
f5_converter.py -f bigip.conf
ls output
```

#### Netscaler Converter Usage
See all the Netscaler converter options
```sh
netscaler_converter.py -h
```
Convert ns.conf into Avi configuration. Output is in output directory
```sh
netscaler_converter.py -f ns.conf
ls output
```

#### virtualservice_examples_api Usage
Create a basic virtualservice named basic-vs
```sh
virtualservice_examples_api.py -h
virtualservice_examples_api.py -c 10.10.25.42 -i 10.90.64.141 -o create-basic-vs -s 10.90.64.12
```
