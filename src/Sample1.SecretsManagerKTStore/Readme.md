# AWS Lambda Empty Function Project (with Kerberos + MSSQL)

> This is a clone of the stock `lambda.EmptyFunction` starter project that's been augmented
> to include the use of `KerberosManager` and to invoke a simple command to
> a target SQL Server (MSSQL) database using _integrated authentication_.

Instead of pulling the KeyTab file from S3, it pulls it from AWS Secrets Manager, which supports the native binary format.

It also takes a CSV list of KDC's (instead of a single KDC) so that if the primary KDC is down for maintenance, the Lambda will use a fallback KDC to initialize.

It requires the following environment variables be set:
- KerberosRealm - The fully qualified domain name, e.g. "EXAMPLE.COM"
- KerberosPrincipal - The UPN of your user e.g. "sample_user@EXAMPLE.COM"
- KerberosKeytabSecretId - The name or ARN of the secret that contains your keytab
- SqlServer - The fully qualified DNS name of your SQL Server e.g. "mssql1.example.com"
- KerberosRealmKdcCSV - A CSV list of domain controllers e.g. "DC1.EXAMPLE.COM,DC2.EXAMPLE.COM"
-- If an error is encountered during initialization to the first KDC (e.g. the server is down for maintenance) then the next server in the list will be used.

This starter project consists of:
* Function.cs - class file containing a class with a single function handler method
* aws-lambda-tools-defaults.json - default argument settings for use with Visual Studio and command line deployment tools for AWS

You may also have a test project depending on the options selected.

The generated function handler is a simple method accepting a string argument that returns the uppercase equivalent of the input string. Replace the body of this method, and parameters, to suit your needs. 

## AWS Secrets Manager setup
Once you've generated the keytab file, this is a simple PowerShell Command.

**NOTE**: This assumes that you have already set up your environment with AWS Credentials (see here https://docs.aws.amazon.com/powershell/latest/userguide/specifying-your-aws-credentials.html)

```
Import-Module AWSPowerShell

$secretParams = @{
	SecretId = 'TheNameOfYourSecret'
	SecretBinary = [IO.File]::ReadAllBytes('C:\path\to\keytab\file')
	Region = 'us-west-2' # Or whatever AWS Region that you want to write to
	VersionStage = 'AWSCURRENT'
}

Write-SECSecretValue @secretParams
```

The Lambda function role will need the following IAM Policy attached in order to access the keytab in Secrets Manager:
```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": "secretsmanager:GetSecretValue",
			"Resource": "the ARN of your keytab secret (this value is returned from the script above)"
		}
	]
}
```

## Here are some steps to follow from Visual Studio:

To deploy your function to AWS Lambda, right click the project in Solution Explorer and select *Publish to AWS Lambda*.

To view your deployed function open its Function View window by double-clicking the function name shown beneath the AWS Lambda node in the AWS Explorer tree.

To perform testing against your deployed function use the Test Invoke tab in the opened Function View window.

To configure event sources for your deployed function, for example to have your function invoked when an object is created in an Amazon S3 bucket, use the Event Sources tab in the opened Function View window.

To update the runtime configuration of your deployed function use the Configuration tab in the opened Function View window.

To view execution logs of invocations of your function use the Logs tab in the opened Function View window.

## Here are some steps to follow to get started from the command line:

Once you have edited your template and code you can deploy your application using the [Amazon.Lambda.Tools Global Tool](https://github.com/aws/aws-extensions-for-dotnet-cli#aws-lambda-amazonlambdatools) from the command line.

Install Amazon.Lambda.Tools Global Tools if not already installed.
```
    dotnet tool install -g Amazon.Lambda.Tools
```

If already installed check if new version is available.
```
    dotnet tool update -g Amazon.Lambda.Tools
```

Execute unit tests
```
    cd "BlueprintBaseName/test/BlueprintBaseName.Tests"
    dotnet test
```

Deploy function to AWS Lambda
```
    cd "BlueprintBaseName/src/BlueprintBaseName"
    dotnet lambda deploy-function
```
