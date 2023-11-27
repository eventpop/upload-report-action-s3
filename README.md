# upload-report-action-s3

A GitHub Action to upload HTML report to S3 with OIDC-based authentication.

**Use case:** You have generated an HTML report of your build process (e.g. [Playwright](https://playwright.dev/docs/trace-viewer-intro#opening-the-html-report), [Allure](https://docs.qameta.io/allure/#_overview_page), coverage report, etc.) and you want to publish it somewhere that people can easily view.

This action does the following:

1. [Logs in to AWS via OIDC](https://github.com/aws-actions/configure-aws-credentials) using the configuration specified in `aws-role` and `aws-region`. Thanks to [GitHub and AWS’ OIDC support](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services), we don’t have to put in access key secrets in our workflow.
2. Generates a random and unguessable URL for the report.
3. Configures AWS CLI so that it uploads files faster. This is because test reports tends to have many files.
4. Uploads the files specified in the directory `path` (this directory is expected to have an `index.html` file) to the S3 bucket `s3-bucket` (this bucket is expected to be accessible via a URL specified in `view-url`).
5. Generates a [job summary](https://github.blog/2022-05-09-supercharging-github-actions-with-job-summaries/) with a link to the report file.

## Usage

<details><summary>Initial AWS setup</summary>

This only has to be set up once once.

1. Create an S3 bucket. Make sure the bucket is accessible. You can set up a lifecycle policy to delete old files after some time (e.g. 3 months).
2. Create an IAM Role and OIDC Provider. [You can use this CloudFormation template](https://github.com/aws-actions/configure-aws-credentials?tab=readme-ov-file#sample-iam-oidc-cloudformation-template).
3. Grant that role access to the S3 bucket.

</details>

In the project that needs to publish a report:

1. Make sure the GitHub Actions have the `id-token: write` permission.

   ```yaml
   permissions:
     id-token: write
     contents: read
   ```

2. Use the action:

   ```yaml
   - name: Publish report
     if: always()
     uses: eventpop/upload-report-action-s3@main
     with:
       aws-role: arn:aws:iam::000000000000:role/github-oidc-Role-AAAAAAAAAAAA
       aws-region: ap-southeast-1
       s3-bucket: my-bucket
       view-url: https://my-bucket.s3.ap-southeast-1.amazonaws.com
       name: my-project-report
       path: allure-report
   ```

> [!TIP]
> At Eventpop, we have a private composite action that calls into this public action with these inputs hardcoded: `aws-role`, `aws-region`, `s3-bucket` and `view-url`. This allows Eventpop projects to more conveniently publish reports by just specifying the `name` and `path`.
