Deploying a Static Site with GitHub Actions
Introduction
You are working for a consumer software company that is preparing to announce a major new product. You have been tasked with setting up the landing page for users to learn more and sign up for the beta. Marketing has approved an initial static JavaScript page, but this page will need to be able to update quickly as news breaks. You will need to set up a GitHub Actions workflow to build and deploy the site to a non-Production bucket, and then promote it to a Production bucket once Marketing has approved the new design.

Solution
Log in to the AWS Management Console using the credentials provided in the lab.

If you don't already have a personal GitHub account, create one before you start the lab by signing up at github.com.

Note: The solution code for this lab is provided in the GitHub repository's solution branch.

You'll also need to use a code editor, like Visual Studio Code, for this lab. Make sure you have one installed before starting this lab.

Set Up Secrets in GitHub
Add an IAM User and Generate Keys in the AWS Console
In the AWS Management Console, navigate to S3 using the Services menu or the unified search bar. You should see 2 public buckets provided for the lab: a non-Production bucket and a Production bucket.
Note both bucket names. You'll need them later on in the lab.
Navigate to IAM using the Services menu or the unified search bar.
Select Users from the IAM dashboard, then click Add users.
Fill in the user details:
In the User name field, enter github-actions-user.
In the Access type field, select Programmatic access.
Click Next: Permissions.
Set the user permissions:
Below Set permissions, select Attach existing policies directly.
In the policy search bar, search for AmazonS3FullAccess.
Check the box to select the AmazonS3FullAccess policy.
Click Next: Tags. You don't need to add any tags for this lab.
Click Next: Review.
Review the user details, then click Create user. AWS generates an access key ID and a secret access key.
Note the access key ID and the secret access key. You'll need them later on in the lab.
Note: Make sure you store these keys before closing out of the window, because you won't be able to access them again.

After noting the keys, click Close to close out of the window.
Set Up IAM Keys as Secrets in GitHub
Open the GitHub repository in a new tab using the link provided in the lab resources.
Click Fork in the upper right corner, then select your personal GitHub account to fork the repo into your own account.
Select Settings in the top menu bar.
Select Secrets in the side bar menu.
Create a secret for the access key ID:
Click New repository secret.
In the Name field, enter AWS_ACCESS_KEY_ID.
In the Value field, paste your saved access key ID.
Click Add secret.
Create a secret for the secret access key:
Click New repository secret.
In the Name field, enter AWS_SECRET_ACCESS_KEY.
In the Value field, paste your saved secret access key.
Click Add secret.
Set Up a Non-Production Workflow
Create the Non-Production Workflow
Clone your forked GitHub repository to your local workstation using a code editor of your choice (like Visual Studio Code).
Note: For this lab guide, we'll provide instructions based on the Visual Studio Code interface.

Open the code editor's terminal.
Create a new branch:
git checkout -b feature-initial-page
In the code editor's side bar menu, click the New File icon to add a non-Production workflow file.
In the file location text box, enter .github/workflows/nonproduction.yaml.
.github/workflows is the directory, and nonproduction.yaml is the file name.

In the nonproduction.yaml file editor, configure the workflow:

name: Deploy feature branches to nonproduction bucket

on:
  push:
    branches:
      - feature*

jobs:

  build:
    runs-on: ubuntu-latest
    env:
      BUCKET_NAME: <Nonproduction bucket name here>
    steps:
      - name: Check out code
        uses: actions/checkout@v2
      - name: configure aws cli
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      - name: set up node.js
        uses: actions/setup-node@v2
        with:
          node-version: 14
      - name: build site
        run: |
          npm ci
          npm run build
      - name: deploy files to bucket
        run: aws s3 cp public s3://${{ env.BUCKET_NAME }} --recursive --acl public-read
Note: Be sure to replace <Nonproduction bucket name here> with the name of your non-Production S3 bucket.

This code triggers the workflow on push events for only feature branches. The job runs on a generic Linux runner. First, it checks out your repo code. Next, it configures the AWS CLI on the runner, using your stored secrets. Then, it sets up node.js to build your static site. Finally, it uploads your files to the non-Production S3 bucket.

In the code editor's SSH terminal, commit your changes to nonproduction.yaml:

git add --a
git commit -m 'created nonproduction workflow'
After you commit your changes, push them:

git push --set-upstream origin feature-initial-page
Verify the Workflow Executed Successfully
Navigate back to your forked GitHub repository.
Select Actions in the top menu bar. You should see that GitHub is executing your non-Production workflow.
After the workflow executes successfully, navigate back to S3 in the AWS Management Console.
Select the non-Production bucket name. You should see that files have been added to the bucket.
Select the Properties tab and scroll down to Static website hosting.
Open the static site's URL in a new tab. You should see your non-Production website has been created.
Set Up a Production Workflow
Create the Production Workflow
Navigate back to your code editor.
In the code editor's side bar menu, click the New File icon to add a Production workflow file. You'll add this file to the .github/workflows directory.
In the file name text box, enter production.yaml.
In the production.yaml file editor, configure the workflow:
name: Deploy site to production bucket

on:
  push:
    branches:
      - main

jobs:

  build:
    runs-on: ubuntu-latest
    env:
      BUCKET_NAME: <Production bucket name here>
    steps:
      - name: Check out code
        uses: actions/checkout@v2
      - name: configure aws cli
        uses: actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      - name: set up node js
        uses: actions/setup-node@v2
        with:
          node-version: 14
      - name: build site
        run: |
          npm ci
          npm run build
      - name: deploy files to bucket
        run: aws s3 cp public s3://${{ env.BUCKET_NAME }} --recursive --acl public-read
Note: Be sure to replace <Production bucket name here> with the name of your Production S3 bucket.

In the code editor's SSH terminal, save and commit your changes to production.yaml:
git add --a
git commit -m 'added production workflow'
After you commit your changes, push them:
git push
Verify the Workflow Executed Successfully
Navigate back to your forked GitHub repository.
Select Actions in the top menu bar. You should see that the workflow triggered your non-Production job, and this is because the Production workflow is still on the feature branch instead of the main branch.
Select Pull requests in the top menu bar, then click New pull request.
Configure the pull request:
Use the base repository and head repository dropdowns to select your forked repo if needed.
Ensure the base dropdown is set to main.
Use the compare dropdown to select the feature-initial-page branch.
Click Create pull request.
Click Merge pull request, then click Confirm merge.
After the pull request is successfully merged and closed, you can click Delete branch to keep your working tree clean.
Select Actions in the top menu bar. You should now see that GitHub is executing your Production workflow on the main branch as expected.
After the workflow executes successfully, navigate back to S3 in the AWS Management Console.
Select the Production bucket name.
Select the Properties tab and scroll down to Static website hosting.
Open the static site's URL in a new tab. You should see your Production site has been created, and it looks the same as the non-Production site.
Update the Website with an Announcement
Navigate back to your code editor.
Switch back to your main branch:
git checkout main
Pull down all the changes from the merge you completed in GitHub:
git pull
Create a branch for new features:
git checkout -b feature-launch-date
In the code editor's side bar menu, expand the src folder.
Open the pages folder, then select the index.js file.
In the index.js file editor, add an <h4> code block below the <h1> </h1> code block:
<h4 className='title'>Launch Date Announced!!!</h4>
In the code editor's SSH terminal, commit your changes:
git add --a
git commit -m 'added launch date header'
After you commit your changes, push them:
git push --set-upstream origin feature-launch-date
Verify the Website Updates
Navigate back to your forked GitHub repository and verify the workflow update triggered an action. Since this update is for a feature branch, it should have triggered your non-Production workflow.
After the workflow executes successfully, navigate back to your non-Production site.
Refresh the page. You should see the site updated with your announcement.
Navigate back to your forked GitHub repository. You'll open another pull request to apply your update to your Production site.
Select Pull requests in the top menu bar, then click New pull request.
Configure the pull request:
Use the base repository and head repository dropdowns to select your forked repo if needed.
Ensure the base dropdown is set to main.
Use the compare dropdown to select the feature-launch-date branch.
Click Create pull request.
Click Merge pull request, then click Confirm merge.
After the pull request is successfully merged and closed, you can click Delete branch.
Select Actions in the top menu bar. You should now see that GitHub is executing your Production workflow on the main branch.
After the workflow executes successfully, navigate back to your Production site.
Refresh the page. You should see the site updated with your announcement.
Conclusion
Congratulations — you've completed this hands-on lab!