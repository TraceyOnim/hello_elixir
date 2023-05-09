## Deploying to Fly.io

Ensure [flyctl](https://fly.io/docs/hands-on/install-flyctl/) command-line is installed, it will enable you to work with Fly.io. 

Fly.io enables you to deploy using Docker image. Fly has a pre-built docker image `flyio/hellofly:latest` 


### Steps

1. fly launch --image flyio/hellofly:latest
  - fly launch creates docker <fieldset></fieldset>
2. fly deploy
    - Deploy the application after making changes 
3. fly status
    - app deployment details
3. fly open 
    - to see your deployed app in action
    - opens the app on http version
    - use `fly open/ name` to upgrade to https

4. fly logs
    - tail your application logs
5. fly status -a hello_elixir-db 
    - Database deployment details 

6. fly secrets 
    - to configure secrets
    i.e `fly secrets set MY_SECRET_KEY=my_secret_value`





### SSH to your application

(consult more here)

https://fly.io/docs/elixir/the-basics/iex-into-running-app/

Use `SSH` to login to or run commands on `VMs`.

`flyctl ssh [command][flags]`

Establish an SSH Shell to our machine on Fly.io.This step sets up a root certificate for your account and then issues a certificate.

 `fly ssh issue --agent`




#### commands

1. [console](https://fly.io/docs/flyctl/ssh-console/)
- Connect to a running instance of the current app
  `flyctl ssh console [flags]`


### Naming Elixir Node
In your Elixir application, run this command:

`mix release.init`

Then edit the generated rel/env.sh.eex file and add the following lines:

```
ip=$(grep fly-local-6pn /etc/hosts | cut -f 1)
export RELEASE_DISTRIBUTION=name
export RELEASE_NODE=$FLY_APP_NAME@$ip

```

### Clustering your application
There are 2 parts to getting clustering quickly setup on Fly.

   1. Installing and using libcluster

   ```
   defp deps do
    [{:libcluster, "~> MAJ.MIN"}]
   end
   ```

   ```
   lib/hello_elixir/application.ex

   defmodule HelloElixir.Application do
  use Application

  def start(_type, _args) do
    topologies = Application.get_env(:libcluster, :topologies) || []

    children = [
      # ...
      # setup for clustering
      {Cluster.Supervisor, [topologies, [name: HelloElixir.ClusterSupervisor]]}
    ]

    # ...
  end

  # ...
end

   ```

Add topologies configuration

   ```
   config/runtime.exs

     app_name =
    System.get_env("FLY_APP_NAME") ||
      raise "FLY_APP_NAME not available"

  config :libcluster,
    debug: true,
    topologies: [
      fly6pn: [
        strategy: Cluster.Strategy.DNSPoll,
        config: [
          polling_interval: 5_000,
          query: "#{app_name}.internal",
          node_basename: app_name
        ]
      ]
    ]

   ```
 
   2. Scaling our application to multiple VMs 

      There are two ways to run multiple VMs.

    a. Scale our application to have multiple VMs in one region.

      `fly scale count 2`

    b. Add a VM to another region (multiple regions). 

       https://fly.io/docs/elixir/the-basics/clustering/#scaling-to-multiple-regions


### The cookie situation
