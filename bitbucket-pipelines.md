# Bitbuket pipelines, remote ssh, purging the cache on the Cloudfront and callign custom .sh script

## Bitbucket secured properties

- $SSH_PRIVATE_KEY -> base64 decoded private key used to connect to the ssh server

- ${BITBUCKET_COMMIT} and ${BITBUCKET_BUILD_NUMBER} is native properties in the pipelines used to build the version

- $CF_API_EMAIL - your logging email to the cloudfront

- $CF_API_KEY - API key from cloudfront

bitbucket-pipelines.yml file looks like:

```yml
image: atlassian/default-image:latest

pipelines:
  default:
    - step:
        deployment: production
        script:
          - mkdir -p ~/.ssh
          - echo $SSH_PRIVATE_KEY | base64 --decode -i > ~/.ssh/id_rsa
          - chmod 400 ~/.ssh/id_rsa
          - cat ./deploy.sh | ssh -T deploy@37.157.193.23
          - ssh -T <user>@<server_IP> "touch <path_to_web_app>/version.txt | echo "${BITBUCKET_COMMIT}_${BITBUCKET_BUILD_NUMBER}" > <path_to_web_app>/version.txt"
          - echo "Deploy step finished now"
          - curl -X POST "https://api.cloudflare.com/client/v4/zones/<your_zone>/purge_cache" -H "X-Auth-Email:$CF_API_EMAIL" -H "X-Auth-Key:$CF_API_KEY" -H "Content-Type:application/json" --data '{"purge_everything":true}'
```

In this i'm using the content of the version.txt as a information in the APP to display the build version.
So it's up to you to leave it, change it or remove it.


deploy.sh file looks like:

```sh
echo "Deploy script started"
cd <path_to_web_app>
sh pull.sh
echo "Deploy script finished execution"
```

pull.sh file looks like:

```sh
git fetch --all
git reset --hard origin/master # to reset all manual changes on the server ;)
## ... do some stuff what you need
echo "Deploy finished"
```


