# Hugo + Forestry + CircleCI + RSync/AWS S3

This repo serves as example of how to handle a Hugo deployment using CircleCI in conjunction w/ Forestry, using rsync for hosted servers or the `awscli` for sites hosted on S3.

This method could also be applied to:
- Other static site generators (Jekyll, etc)
- Other deployment sources (Heroku, Zeit, Google Cloud Storage, Google Firebase, and much more!)

## Getting Started
This repository assumes you already have a working server or an S3 bucket with an IAM user ready-to-use.

1. Fork this repository; it contains everything you need to get started.
2. Sign or login to [Forestry.io](https://forestry.io)
3. Signup or login to [CircleCI](https://circleci.com)

### Setup Forestry
1. Set up a new site, using the fork as the source
2. Set the "Hosting Connection" to "Commit to source repo only"
   **This connection setting allows Forestry to commit any change made to your project to the repository"
3. You're done!

### Setup CircleCI
1. Login to CircleCI, and connect to GitHub if necessary
2. Go to the "Projects" page, click "Add Project", and select the fork from your repositories
3. Click "Build" -- this build will fail, because you haven't set up your build!

## Setup your build

### Deploying to S3
Before you can setup deployment to your own S3 bucket, you must provide your authentication details to CircleCI.

1. In CircleCI, navigate to "Builds", and press the "Cog" icon beside your fork repository
2. Then navigate to "AWS Permissions" in the sidebar
3. On this page, provide Access Key ID and Secret Key associated with your S3 IAM user.
   *Note, it's recommended to setup a separate [IAM machine-user](http://docs.aws.amazon.com/general/latest/gr/root-vs-iam.html) for use with CircleCI*

Now we can configure CircleCI. Head over to GitHub and open up `.circleci/config.yml` in the GitHub editor.

You'll need to change a few lines:

1. Update `{{ s3_bucket }}` to the name of your bucket
2. Update `{{ s3_path}}` to your desired path (or remove it for base path)
2. Remove the 3 lines starting at `# rsync example`
4. That's it! Hit commit and CircleCI will rebuild this Hugo project and deploy it to the bucket.

```
- deploy:
          name: deploy
          command: |
            if [ "${CIRCLE_BRANCH}" = "master" ]; then
              # aws s3 example
              aws s3 sync $HUGO_BUILD_DIR s3://example.forestry.io/ --delete

              # rsync example
              rsync -av --progress -e "ssh -o StrictHostKeyChecking=no" $HUGO_BUILD_DIR ec2-user@52.91.107.33:/home/ec2-user/
            else
              echo "Not master branch, dry run only"
            fi
```

### Deploying to A Server With Rsync
Before you can setup deployment to your server, you must provide your SSH authentication details to CircleCI.

1. In CircleCI, navigate to "Builds", and press the "Cog" icon beside your fork repository
2. Then navigate to "SSH Permissions" in the sidebar
3. On this page, click "Add SSH Key" and paste in the contents of your Private Key.
4. If desired, set hostname to the IP/hostname of your server for additional security.

Now we can configure CircleCI. Head over to GitHub and open up `.circleci/config.yml` in the GitHub editor.

You'll need to change a few lines:

1. Update `{{ ssh_user }}` to the user associated with your SSH key
2. Update `{{ ssh_hostname }}` to your hostname
3. Update `{{ ssh_path }}` to the desired path on your server.
2. On line 23, update `{{ ssh_fingerprint }}` to the fingerprint of your SSH key
   *This can be found on the "SSH Permissions" page in CircleCI*
2. Remove 3 lines starting at `# aws s3 example`
3. That's it! Hit commit and CircleCI will rebuild this Hugo project and deploy it to the bucket.

## CircleCI Configuration
This breaks down some of the custom configuration for this build workflow. 

For a full rundown on how CircleCI's `config.yml` works, see the [sample config docs](https://circleci.com/docs/2.0/sample-config/).





