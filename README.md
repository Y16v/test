# devops-prow
Prow is a kubernetes based ci/cd tool. For more go to this link https://github.com/kubernetes/test-infra/blob/master/prow/getting_started_deploy.md
## How to deploy Prow ?
#### Github configuration
Before deploing Prow, ensure that you have created access token in Github. Go to Profile __Settings > Developer settings > Personal access token__ and check the boxes: `admin:org, admin:org_hook, admin:repo_hook, and repo`. After that access token will be generated. Copy the new token an save it somewhere because it will be necessary later. In addition, to talk with Github you will need another token which is `hmac-token`. hmac-token is the token that you give to Github for validating webhooks. Genarate it using any randomness-generator. For example,
```bash
openssl rand -hex 20 > /path/to/hook/secret  
```
Under the `aws/secrets` you can find two json files related to github tokens. Insert personal access and hmac tokens to corresponding files `github-access-token.json` and `github-hmac-token.json`. To clone private repo you need to create a ssh key and save to the `ssh-secret.json` file. Note that you should paste all the content of your ssh key from `-----BEGIN RSA PRIVATE KEY-----` till `-----END RSA PRIVATE KEY-----`. As an example take a look to the current `ssh-secret.json` file.

#### Logs persistency
As a storage for prow logs we use S3 bucket in each cluster. Before storing in the bucket we created a user `prow` with a permission to S3 service and store user's credentials such as `access_key` and `secret_key` under the `aws/secrets/clusters/cluster_name/s3-credentials.json`. However, there might be another way to persist logs in s3 like using a role. Unfortunately, I don't have enough time to research on it. After the secret's creation we should specify the secret name in `config.yaml` file.

#### Prow configuration
As previously mentioned that prow has microservice architecture. That's why it has several components that must be deployed. To configure all of these components prow has a `config.yaml` file under the `kubernetes/clusters` folder. Moreover, in the `config.yaml` file we can describe periodic jobs, which will run at specified time.

#### Jobs
To triger any jobs we should do two steps: add a webhook to github, and define jobs that will be triggered after github events like push. You must be an admin to have a permission to add any webhooks to repo. In a repo go to __Settings > Webhooks__ and generate them. Paste a DNS name of prow, change content type to `application/json`, paste `hmac-token` and select `Send me everything`. Wait until a green tick will appear near generated token. After that go to the repo that should be build in prow then add `.prow.yaml` file. This file describes the jobs on which conditions prow would start a build. __Note: to clone any private repo we have to add paramaters like: clone_uri, ssh_secrets_key.__ For more info about job's definition click on this link https://github.com/kubernetes/test-infra/blob/master/config/jobs/README.md 

#### Tide configuration
Prow has the `tide` component, which are used for managing a pool of github's PRs that match a given set of criteria. Tide is able to retest and merge PRs. To met the tide's criteria in the `config.yaml` file add labels in a `query` parameter. By using this paramater tide will check PRs. You should specify repos that are related to clusters. For more in info go to this link https://github.com/kubernetes/test-infra/tree/master/prow/tide .To use labels like `lgtm` or `approved` we have to install related plugins. You can check it in `plugins.yaml` file. __Note: now, only the `lgtm` label works.__ I have tried with the `approved` label but it was unsuccessful. However, there is an example https://raw.githubusercontent.com/falcosecurity/test-infra/master/config/plugins.yaml, where other users have configured plugins. Maybe you should look on it to configure plugins.

#### Presets
Now instead of using additional container to build docker image like in jenkins, prow uses a preset and script. The script name is `tron-prow-runner` and it locates under the `bin` folder in `bittorrent/devops-images-tools`

*Useful links*
- https://github.com/kubernetes/test-infra/tree/master/prow
- https://aws.amazon.com/blogs/opensource/how-falco-uses-prow-on-aws-for-open-source-testing/
- https://github.com/falcosecurity/test-infra
