# ref
- http://docs.pivotal.io/platform-automation/v2.1/reference/pipeline.html#installing-ops-manager-and-tiles


## config
- docs: http://docs.pivotal.io/platform-automation/v2.1/reference/inputs-outputs.html
- sample: https://github.com/myminseok/platform-automation-configuration-template
```
platform-automation-configuration-template
└── dev-1
    ├── config
    │   ├── auth.yml    
    │   └── opsman-2.4.yml
    ├── download-product-configs
    ├── env
    │   └── env.yml
    ├── generated-config
    ├── state
    └── vars

```

### env.yml
- http://docs.pivotal.io/platform-automation/v2.1/configuration-management/configure-env.html
```
---

# Env Config
# This file contains properties for targeting and logging into the Ops Manager API.

target: ((opsman_target))
connect-timeout: 30                 # default 5
request-timeout: 3600               # default 1800
skip-ssl-validation: true           # default false
username: ((opsman_admin.username))
password: ((opsman_admin.password))
decryption-passphrase: ((decryption-passphrase))
```


- opsman-2.4.yml:  http://docs.pivotal.io/platform-automation/v2.1/reference/inputs-outputs.html#vsphere
- auth.yml : http://docs.pivotal.io/platform-automation/v2.1/configuration-management/configure-auth.html


## pipeline
- sample: https://github.com/myminseok/platform-automation-pipelines-template

### install-opsman-params.yml
```

foundation: dev-1

s3:
  endpoint: https:///s3.pcfdemo.net
  access_key_id: ((s3_access_key_id))
  secret_access_key: ((s3_secret_access_key))
  region_name: ""
  buckets:
    platform_automation: platform-automation
    foundation: dev-1
    pivnet_products: pivnet-products
    installation: installation

git:
  platform_automation_tasks:
    uri: pivotal@git.pcfdemo.net/platform/platform_automation_tasks.git
  configuration:
    uri: pivotal@git.pcfdemo.net:platform/platform-conf.git
  variable:
    uri: pivotal@git.pcfdemo.net:platform/platform-conf.git
  user: 
    email: ((git_user.email))
    username: ((git_user.username))
  private_key: ((git_private_key.private_key))

credhub:
  server: https://concourse.pcfdemo.net:8844
  ca_cert: ((credhub_ca_cert.certificate))
  client: ((credhub_client.username))
  secret: ((credhub_client.password))
  interpolate_folders: dev-1/config dev-1/env


pivnet: 
  token: ((pivnet_token))

#opsman_image_s3_versioned_regexp:  "*vsphere*.ova"
opsman_image_s3_versioned_regexp:  pcf-vsphere-(.*).ova

```



###  register secret to concourse credhub.
```

credhub set -t value -n /concourse/main/s3_access_key_id -v <S3_ACCESS_KEY>
credhub set -t value -n /concourse/main/s3_secret_access_key -v "<S3_SECRET>"
credhub set -t value -n /concourse/main/pivnet_token -v <YOUR_PIVNET_TOKEN>


credhub set -t value -n /concourse/main/git_user_email -v <GIT_USER_EMAIL>
credhub set -t value -n /concourse/main/git_user_username -v <GIT_USER_NAME>

# register ssh key for git. ex) ~/.ssh/id_rsa
credhub set -t rsa  -n /concourse/main/git_private_key  -p ~/.ssh/id_rsa 
 
cd concourse-bosh-deployment/cluster
bosh int ./concourse-creds.yml --path /atc_tls/certificate > atc_tls.cert
credhub set -t certificate -n /concourse/main/credhub_ca_cert -c ./atc_tls.cert

grep concourse_to_credhub ./concourse-creds.yml
credhub set -t user -n /concourse/main/credhub_client -z concourse_to_credhub -w <concourse_to_credhub>

credhub set -t user  -n /concourse/dev-1/opsman_admin -z admin -w <YOUR_PASSWORD>
credhub set -t value -n /concourse/dev-1/decryption-passphrase -v <YOUR_PASSWORD>
credhub set -t value -n /concourse/dev-1/opsman_target -v https://opsman_url_or_IP


```

## run pipeline

```
fly -t demo sp -p install-opsman -c install-opsman.yml -l ./install-opsman-params.yml

```

