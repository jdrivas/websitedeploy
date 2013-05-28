Stage 3 - Landing Page and Website
==================================
### Static website with prodution and staging deployment to AWS S3

This assumes a `./site` directory that contains a ready to deploy static website. From there it uses `rake` to automate the deployment to either a staging location or the production location. The locations are buckets in a Stage 3 Systems AWS S3 account. To deploy you need to have been given an Amazon IAM user id for the Stage 3 Amazon account. You will also be given credentials that will include two keys, the access-key-id and the secret-key. These numbers are used to connect to Amazon and provide you with permissions to deploy to the site.

Deployment needs two things: 1. Committing any changes to the appropriate git branch (staging, or production); 2. Issuing a rake deploy command on the command line: either `rake deploy_staging` or `rake deploy_production`

## Local Setup & Prerequisites

### Machine setup
Rake is a ruby tool and the Rakefile relies on a few libraries to get the job done, like the Amazon AWS APi for Ruby. If you're on a mac you're almost all set. On the mac `ruby` and `rake` are installed. So is the ruby package manager `gem`. If you're on linux you'll need to ensure you've got a ruby installed. If you're on Windows there is no hope. In addition this assumed you've got `git` installed and can talk to the Stage 3 Systems github account. 

There is additional tool that you will have to install. Bundler is used to specify which ruby packages a particular installation might need. Given that you've got ruby installed you can install bundler with: 

`prompt> gem install bundler`

The final steps are to clone the repo to your machine, run bundler to get the necessary libraries for deployment and start work:

Clone the master repo:

`prompt> git clone git@github:stage3systems/stage3-static-website website_work`

Cd into the repo

`prompt> cd website_work`

Install dependencies (note the command is bundle, not bundler):

`prompt> bundle install`

### Amazon AWS IAM setup & Credential installation
In order to be allowed to install the files onto S3, you need to have both a user with credentials and permissions to do so. 

To obtain user credentials see someone with Amazon AWS administration access. Currently that's Mark Deepwell and David Rivas. They can set you up with a user account and provide you with Amazon credentials. The needed credentials are two password like tokens or keys: aws-access-key-id and aws-secret-access-key. You should keep both of these hidden and safe.

Once you have a set of credentials you need to let the system know you have them. This can be done with a rake command:

```
  prompt> rake add_aws_credentials
  Couldn't find your AWS keys stored locally. I'll do that now.
  Your keys will be stored in the file: .aws_credentials.
  The file should be in your .gitignore as well. Don't change this.
  YOU DO NOT WANT TO STORE YOUR KEYS IN GIT
  AWS ACCESS KEY ID: <your-access-key-id-goes-here>
  AWS Secret Key: <your-secret-access-key-goes-here>
  These will be stored in .aws_credentials in json format.
```

As the output notes, these keys will be stored in the file `.aws_credentials` as json. There should be a `.gitignore` file that excludes this file from being seen by git. When this file is created it should have permissions set to only let the user read/write the file.

## Regular Use

### Git branching and staging vs. production
A deployment happens from the latest version of a git branch. The idea is to make sure that the production website is always the latest version in the production branch of the maseter git repo on github. This gets mirrored with staging and the staging branch of the master github repo. So, prior to deployment you must push to the master repo as that's where the deploy script gets that version of the site to deploy.

You can go about this anyway you want, but the recommended practice is to follow a [gitflow](http://nvie.com/posts/a-successful-git-branching-model/) like branching model without adhering to the precise gitflow conventions. For deployment there are only two required branches: *production* and *staging*. The idea would be to work in a *feature* or *development* branch until it's ready for review, merge the *development* branch into *staging* and deploy to staging for review. Once reviews are over and it's ready to go to live, merge the finished *staging* branch into *production* and deploy to production. On deployment to production the site it's live.

### Deploying
As said above, there are two steps to deployment, first push the appropriate branch to the github master repo, then use rake to deploy.

For Staging this looks like:

```
prompt> git push origin staging

prompt> rake deploy_staging
```

For production:

```
prompt> git push origin production

prompt> rake deploy_production
```

An important detail is that the deploy command brings over a clean version of the appropriate into a temporary directory to do the copy of files over into the S3 file system. The website itself is pretty small, but should you run into problems check to see if you're running out of disk space. 

A better solution for deployment would be to use a git-hook to automatically deploy to S3 whenever a git push was done to either the production or staging branches of the master github repo. Github supports a [mechanism](https://help.github.com/articles/post-receive-hooks) that will send a message to a web-service when a push to the repo has occured. To make use of this we'd have to implement the web-service on a running machine that took the message from github then did the pull from the master repo and then copied the files up to S3. We may do that latter, it would be a nice start to a more general deployment service for Stage 3 Systems.

