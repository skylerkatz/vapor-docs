# Environments

[[toc]]

## Introduction

As you may have noticed, projects do not contain much information or resources themselves. All of your deployments and invoked commands are stored within environments. Each project may have as many environments as needed.

Typically, you will have an environment for "production", and a "staging" environment for testing your application. However, don't be afraid to create more environments for testing new features without interrupting your main staging environment.

## Creating Environments

Environments may be created using the `env` Vapor CLI command:

```bash
vapor env my-environment
```

This command will add a new environment entry to your project's `vapor.yml` file that you may deploy when ready:

```yaml
id: 2
name: vapor-laravel-app
environments:
    production:
        build:
            - 'composer install --no-dev'
    my-environment:
        build:
            - 'composer install --no-dev'
```

In addition to our native runtimes, Vapor supports Docker image deployments. If you would like an environment to use a Docker image runtime instead of the default Vapor runtime, use the `--docker` option when creating your environment:

```bash
vapor env my-environment --docker
```

This command will create a `my-environment.Dockerfile` file in your application's root directory.

## Environment Variables

Each environment contains a set of environment variables that provide crucial information to your application during execution, just like the variables present in your application's local `.env` file.

### Updating Environment Variables

You may update an environment's variables via the Vapor UI or using the `env:pull` and `env:push` CLI commands. The `env:pull` command may be used to pull down an environment file for a given environment:

```bash
vapor env:pull production
```

Once this command has been executed, a `.env.{environment}` file will be placed in your application's root directory. To update the environment's variables, simply open and edit this file. When you are done editing the variables, use the `env:push` command to push the variables back to Vapor:

```bash
vapor env:push production
```

### Vapor Environment Variables

Vapor automatically injects a variety of environment variables based on your environment's configured database, cache, etc. You will not see these environment variables when you manage your environment, and any variables you manually define will override Vapor's automatically injected variables.

:::tip Variables & Deployments

After updating an environment's variables, the new variables will not be utilized until the application is deployed again. In addition, when rolling back to a previous deployment, Vapor will use the variables as they existed at the time the deployment you're rolling back to was originally deployed.
:::

:::warning Environment Variable Limits

Due to AWS Lambda limitations, your environment variables may only be 4kb in total. You should use "secrets" to store very large environment variables.
:::

### Reserved Environment Variables

The following environment variables are reserved and may not be added to your environment:

- _HANDLER
- AWS_ACCESS_KEY_ID
- AWS_EXECUTION_ENV
- AWS_LAMBDA_FUNCTION_MEMORY_SIZE
- AWS_LAMBDA_FUNCTION_NAME
- AWS_LAMBDA_FUNCTION_VERSION
- AWS_LAMBDA_LOG_GROUP_NAME
- AWS_LAMBDA_LOG_STREAM_NAME
- AWS_LAMBDA_RUNTIME_API
- AWS_REGION
- AWS_SECRET_ACCESS_KEY
- AWS_SESSION_TOKEN
- LAMBDA_RUNTIME_DIR
- LAMBDA_TASK_ROOT
- TZ

In addition, environment variables should not contain `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, or `AWS_SESSION_TOKEN` in their names. For example: `MY_SERVICE_AWS_SECRET_ACCESS_KEY`.

## Secrets

Due to AWS Lambda limitations, your environment variables may only be 4kb in total. However, "secrets" may be much larger, making them a perfect way to store large environment variables such as Laravel Passport keys. You may create secrets via the Vapor UI or the `secret` CLI command:

```bash
vapor secret production
```

:::tip Secrets & Deployments

After updating an environment's secrets, the new secrets will not be utilized until the application is deployed again. In addition, when rolling back to a previous deployment, Vapor will use the secrets as they existed at the time the deployment you're rolling back to was originally deployed.
:::

### Passport Keys

Storing Laravel Passport keys is a common use-case for secrets. You may easily add your project's Passport keys as secrets using the `secret:passport` CLI command:

```bash
vapor secret:passport production
```

## Vanity URLs

Each environment is assigned a unique "vanity URL" which you may use to access the application immediately after it is deployed. When serving these URLs, Vapor automatically adds "no-index" headers to the response so they are not indexed by search engines.

In addition, these secret URLs will continue to work even when an application is in maintenance mode, allowing you to test your application before disabling maintenance mode and restoring general access.

## Custom Domains

Of course, as cool as they are, you don't want to share your vanity URL with the world. You likely want to attach your own custom domain to an environment. To do so, you'll first need to create a [DNS zone](./../domains/dns.md) for the domain. Next, you need to create a [certificate](./../domains/certificates.md) for the domain. All Vapor applications are required to utilize SSL. Don't worry, Vapor's certificates are free and automatically renew.

After creating a certificate, you may attach the domain to your environment using the `domain` configuration option in your `vapor.yml` file. When attaching a domain to an environment, it is not necessary to provide the `www` sub-domain:

```yaml
id: 2
name: vapor-laravel-app
environments:
    production:
        domain: example.com
        build:
            - 'composer install --no-dev'
```

During deployment, Vapor will configure the environment to handle requests on this domain. After the deployment is completed, the deployment output will provide you with CNAME records for the domain. These records will point the domain to your environment.

**Of course, Vapor will automatically add these records to a [DNS zone](./../domains/dns.md) for you. So, if you are using Vapor to manage your DNS records, you are not required to manually add these CNAME records to your DNS configuration.** If you are not using Vapor to manage your DNS, you should manually add these CNAME records to your domain's DNS provider.

:::warning Custom Domain Provisioning Time

Due to the nature of AWS CloudFront, custom domains often take 30-45 minutes to become fully active. So, do not worry if your custom domain is not immediately accessible after deployment.
:::

### Multiple Domains

Vapor allows you to attach multiple domains to a single project. Before doing so, ensure you have a valid certificate for each of the domains:

```yaml
id: 2
name: vapor-laravel-app
environments:
    production:
        domain:
            - example1.com
            - example2.com
        build:
            - 'composer install --no-dev'
```

### Wildcard Subdomains

You may attach a domain that supports wildcard subdomains to a Vapor environment if that environment is also using an [application load balancer](./../resources/networks.md#load-balancers) and you have a valid certificate for the domain. To attach a wildcard domain to your environment, specify a `*` as the subdomain:

```yaml
id: 2
name: vapor-laravel-app
environments:
    production:
        domain: '*.example.com'
        build:
            - 'composer install --no-dev'
```

## Maintenance Mode

When deploying a Laravel application using a traditional VPS like those managed by [Laravel Forge](https://forge.laravel.com), you may have used the `php artisan down` command to place your application in "maintenance mode". To place a Vapor environment in maintenance mode, you may use the Vapor UI or the `down` CLI command:

```bash
vapor down production
```

To remove an environment from maintenance mode, you may use the `up` command:

```bash
vapor up production
```

:::tip Maintenance Mode & Vanity URLs

When an environment is in maintenance mode, the environment's custom domain will display a maintenance mode splash screen; however, you may still access the environment via its "vanity URL"
:::

### Customizing The Maintenance Mode Screen

You may customize the maintenance mode splash screen for your application by placing a `503.html` file in your application's root directory.

## Commands

Commands allow you to execute an arbitrary Artisan command against an environment. You may issue a command via the Vapor UI or using the `command` CLI command. The `command` command will prompt you for the Artisan command you would like to run:

```bash
vapor command production

vapor command production --command="php artisan inspire"
```

By default, your command will timeout after one minute. You can configure the timeout of your CLI commands using the `cli-timeout` option within your `vapor.yml` file. This option allows you to specify the maximum number of seconds a CLI command should be allowed to run:

```yaml
id: 2
name: vapor-laravel-app
environments:
    production:
        cli-timeout: 20
        build:
            - 'composer install --no-dev'
```

## Concurrency

By default, Vapor will allow your application to process web requests at max concurrency, which is typically 1,000 requests executing at the same time at any given moment. If you would like to reduce the maximum web concurrency, you may define the `concurrency` option in the environment's `vapor.yml` configuration. Additionally, if you need more than 1,000 concurrent requests, you can submit a limit increase request in the AWS Support Center console.

While maximum performance is certainly appealing, in some situations it may make sense to set this value to the maximum concurrency you reasonably expect for your particular application. Otherwise, a DDoS attack against your application could result in larger than expected AWS costs:

```yaml
id: 2
name: vapor-laravel-app
environments:
    production:
        concurrency: 50
        build:
            - 'composer install --no-dev'
```

## Provisioned Concurrency

By default, when an environment is deployed, the first request it receives may encounter "cold starts". These requests typically incur a penalty of a few seconds while AWS loads a serverless container to serve the request. Once a request has been served by that container, it is typically kept warm to serve further requests with no delay.

To mitigate this issue, Amazon supports "provisioned concurrency". When using this feature, the requested number of execution environments / containers will be provisioned and kept "warm" for you automatically, allowing them to immediately serve any incoming requests. To enable this feature, you may add a `capacity` configuration option to the environment in your `vapor.yml` file:

```yaml
id: 2
name: vapor-laravel-app
environments:
    production:
        capacity: 5
        build:
            - 'composer install --no-dev'
```

For more information on this feature and its pricing, please consult the [AWS documentation](https://aws.amazon.com/blogs/aws/new-provisioned-concurrency-for-lambda-functions/).

:::warning Warming

When using this feature, any `warm` values in your `vapor.yml` file will be ignored.
:::

## Prewarming

By default, when an environment is deployed, the first request it receives may encounter "cold starts". These requests typically incur a penalty of a few seconds while AWS loads a serverless container to serve the request. Once a request has been served by that container, it is typically kept warm to serve further requests with no delay.

To mitigate "cold starts" after a fresh deployment, Vapor allows you to define a `warm` configuration value for an environment in your `vapor.yml` file. The `warm` value represents how many serverless containers Vapor will "pre-warm" by making concurrent requests to the newly deployed application **before it is activated for public accessibility**. Vapor will continue to pre-warm this many containers every 5 minutes while the application is deployed so the specified number of containers are always ready to serve requests:

```yaml
id: 2
name: vapor-laravel-app
environments:
    production:
        warm: 10
        build:
            - 'composer install --no-dev'
```

## Timeout

By default, Vapor will limit web request execution time to 10 seconds. If you would like to change the timeout value, you may add a `timeout` value (in seconds) to the environment's configuration. Note that AWS does not allow Lambda executions to process for more than 15 minutes:

```yaml
id: 2
name: vapor-laravel-app
environments:
    production:
        timeout: 20
        build:
            - 'composer install --no-dev'
```

:::tip Detecting Timeouts In Logs

Remember, you can use the [Vapor UI dashboard package](./../introduction.html#installing-the-vapor-ui-dashboard) to search for timeout occurrences.
:::

## Scheduler

Vapor automatically configures Laravel's task scheduler and instructs it to use the DynamoDB cache driver to avoid overlapping tasks, so no other configuration is required to begin leveraging Laravel's scheduled task feature.

If you would like to disable the scheduler, you may set an environment's `scheduler` option to `false`:

```yaml
id: 2
name: vapor-laravel-app
environments:
    production:
        scheduler: false
        build:
            - 'composer install --no-dev'
```

:::warning Log Messages

Due to Vapor limitations, log messages from scheduled tasks will not appear in AWS CloudWatch or Vapor UI. As a workaround, you should dispatch a queued job from your scheduled tasks and write log messages from your queued job.
:::

## Metrics

A variety of environment performance metrics may be found in the Vapor UI or using the `metrics` CLI command:

```bash
vapor metrics production
vapor metrics production 5m
vapor metrics production 30m
vapor metrics production 1h
vapor metrics production 8h
vapor metrics production 1d
vapor metrics production 3d
vapor metrics production 7d
vapor metrics production 1M
```

### Alarms

You may configure alarms for all environment metrics using the Vapor UI. These alarms will notify you via the notification method of your choice when an alarm's configured threshold is broken and when an alarm recovers.

## Runtime

The `runtime` configuration option allows you to specify the runtime a given environment runs on.

### Native Runtimes

The currently supported native runtimes are `php-7.3`, `php-7.4`, `php-7.4:al2`, `php-8.0`, and `php-8.0:al2`. The runtimes that are suffixed with `al2` use Amazon Linux 2 while those without the suffix use Amazon Linux 1:

```yaml
id: 2
name: vapor-laravel-app
environments:
    production:
        runtime: 'php-8.0:al2'
        build:
            - 'composer install --no-dev'
```

:::warning Amazon Linux 2

Using [Amazon Linux 2](https://aws.amazon.com/amazon-linux-2/) (`php-7.4:al2` or `php-8.0:al2`) is **highly recommended**, as Amazon Linux 1 is no longer maintained as of December 31, 2020.
:::

The following limitations apply to Vapor native runtimes:

- The application size, including the runtime itself, must not exceed 250MB.
- Additional PHP extensions or libraries (such as `imagick`) can not be installed.

### Docker Runtimes

Docker based runtimes allow you to package and deploy applications up to 10GB in size and allow you to install additional PHP extensions or libraries by updating the environment's corresponding `.Dockerfile`.

If you would like to use a Docker image instead of the Vapor native runtimes, set the `runtime` configuration option to `docker` within your `vapor.yml` file:

```yaml
id: 2
name: vapor-laravel-app
environments:
    production:
        runtime: docker
        build:
            - 'composer install --no-dev'
```

:::warning Migrating Existing Environments To A Docker Runtime

When migrating an existing environment to a Docker runtime, please keep in mind that you won't be able to revert that environment to the default Vapor Lambda runtime later. For that reason, you may want to create an environment for testing the Docker runtime first.
:::

For every new Docker based environment, Vapor adds a `.Dockerfile` file that uses one of Vapor's base images as a starting point for building your image. All of Vapor's Docker images are based on Alpine Linux. For example, here's how you may install the FFmpeg library within your custom Docker image:

```docker
FROM laravelphp/vapor:php74

RUN apk --update add ffmpeg

COPY . /var/task
```

Vapor will build, tag, and publish the image on your next deployment; therefore, you should ensure that you have installed [Docker](https://docs.docker.com/get-docker/) on your local machine.

Vapor's base Docker images are:

- `laravelphp/vapor:php74`
- `laravelphp/vapor:php80`

## Gateway Versions

By default, Vapor routes HTTP traffic to your serverless applications using AWS API Gateway. Your application may run on either API Gateway 1.0 or API Gateway 2.0 (HTTP APIs). By default, applications deploy using API Gateway 1.0 as it provides a fuller feature set such as wildcard domains, automatic HTTP to HTTPS redirection, and more.

However, API Gateway 2.0 offers a cost reduction per million requests to your application ($1.00 per million vs. $3.50 per million). If you would like to use API Gateway 2.0, you may specify the `gateway-version` configuration option for a given environment in your `vapor.yml` file:

```yaml
id: 2
name: vapor-laravel-app
environments:
    production:
        gateway-version: 2
        build:
            - 'composer install --no-dev'
```

### HTTP to HTTPS Redirection With API Gateway 2.0

If you choose to use API Gateway 2.0 and would like to support HTTP to HTTPS redirection, we currently suggest using [Cloudflare](https://cloudflare.com) as an external DNS provider for your Vapor application. Cloudflare not only provides DNS, but serves as a reverse proxy to your application and features an option for automatic HTTP to HTTPS redirection.

After a Vapor deployment is completed, Vapor will provide you with CNAME records for the domain(s) associated with your environment. These records will point the domain to your Lambda application. You should manually add these values as CNAME records within the Cloudflare DNS dashboard.

In addition, when using Cloudflare, you should set your Cloudflare SSL / TLS mode to "Full". The "Always Use HTTPS" configuration option may be found under the SSL / TLS menu's "Edge Certificates" tab.

## Custom VPCs

If you want to place your Lambda functions within a VPC that is not managed by Vapor, you may specify the `subnets` and `security-groups` configuration options for a given environment in your `vapor.yml` file:

```yaml
id: 2
name: vapor-laravel-app
environments:
    production:
        subnets:
            - subnet-08aczain4man8bba7
            - subnet-08acr4nd0mbba7
        security-groups:
            - sg-0cr4and0m45b7e0
```

## Deleting Environments

Environments may be deleted via the Vapor UI or using the `env:delete` CLI command:

```bash
vapor env:delete testing
```
