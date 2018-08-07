## AWS CodePipline Actions for AWS CloudFormation

This module contains an Action that allows you to use CloudFormation actions from CodePipeline.

Example usage:

```ts
import codecommit = require('@aws-cdk/aws-codecommit');
import codecommitCodepipeline = require('@aws-cdk/aws-codecommit-codepipeline');
import codepipeline = require('@aws-cdk/aws-codepipeline');
import iam = require('@aws-cdk/aws-iam');
import cdk = require('@aws-cdk/cdk');
import cloudFormationCodePipeline = require('@aws-cdk/aws-cloudformation-codepipeline');

const pipeline = new codepipeline.Pipeline(stack, 'DeploymentPipeline');

const changeSetExecRole = new iam.Role(stack, 'ChangeSetRole', {
    assumedBy: new cdk.ServicePrincipal('cloudformation.amazonaws.com'),
});

/** Source! */
const repo = new codecommit.Repository(stack, 'MyVeryImportantRepo', { repositoryName: 'my-very-important-repo' });

const sourceStage = new codepipeline.Stage(pipeline, 'source');

const source = new codecommitCodepipeline.PipelineSource(sourceStage, 'source', {
    artifactName: 'SourceArtifact',
    repository: repo,
});

/** Deploy! */
const prodStage = new codepipeline.Stage(pipeline, 'prod');
const stackName = 'AStack';
const changeSetName = 'AChangeSet';

new cloudFormationCodePipeline.CreateReplaceChangeSet(prodStage, 'MakeChangeSetProd', {
    stackName,
    changeSetName,
    roleArn: changeSetExecRole.roleArn,
    templatePath: new codepipeline.ArtifactPath(source.artifact, 'some_template.yaml'),
});

new cloudFormationCodePipeline.ExecuteChangeSet(prodStage, 'ExecuteChangeSetProd', {
    stackName,
    changeSetName,
});
```

See [the AWS documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/continuous-delivery-codepipeline.html)
for more details about using CloudFormation in CodePipeline.
