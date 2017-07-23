title: Maven AWS AccessDenied Issue
date: 2017-06-23 11:53:35
tags: 
- Maven
- AWS
---

When I used mvn to package some java codes as usually, there happened a very weird exception:

``` shell
INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 28.410 s
[INFO] Finished at: 2017-06-23T08:29:33+00:00
[INFO] Final Memory: 50M/612M
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal net.alchim31.maven:scala-maven-plugin:3.2.0:compile (scala-compile-first) on project analysis-spark: Execution scala-compile-first of goal net.alchim31.maven:scala-maven-plugin:3.2.0:compile failed: Status Code: 403, AWS Service: Amazon S3, AWS Request ID: 7967A1610B14A2BD, AWS Error Code: AccessDenied, AWS Error Message: Access Denied -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/PluginExecutionException
[ERROR]
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :analysis-spark
```

The exception said that AWS access denied, as we use AWS in our maven settings like belowed:

``` xml
<settings>
   <servers>
    <server>
           <id>maven-s3-release-repo</id>
           <username>USERNAME</username>
           <password>PASSWORD</password>
     </server>
</settings>
```

And I have double checked the authentication information is correct while other developers can package successfully in their own computers.

Finally I found that the problem is because I set a environment variable in Mac:

``` shell
$env | grep AWS
AWS_SECRET_ACCESS_KEY=XXXXX
AWS_ACCESS_KEY_ID=XXXX
```

After unset them, the package procedure became okay. I think that is because the ENV has higher priority than the configuration in `~/.m2/settings.xml`.