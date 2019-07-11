[![Build Status](https://dev.azure.com/jonhoo/jonhoo/_apis/build/status/rusty-pipes?branchName=master)](https://dev.azure.com/jonhoo/jonhoo/_build/latest?definitionId=2&branchName=master)
<!-- [![Codecov](https://codecov.io/github/jonhoo/rusty-pipes/coverage.svg?branch=master)](https://codecov.io/gh/jonhoo/rusty-pipes) -->

Azure _loves_ to try to get you to sign in to GitHub using OAuth,
thereby giving them access to all your public _and private_ repos. This
makes me sad. Here's how you do it "the new way" instead.

First, make sure you have the Azure Pipelines GitHub Application installed:

 - Install the GitHub Azure Pipelines app: https://github.com/apps/azure-pipelines
 - Click "Configure"
 - Click the user or organization you want CI for
 - Towards the bottom, either choose "All repositories" or select the
   repository you want CI for

Then, make sure you have an Azure Project for your GitHub organization:

 - "Create project" over at https://dev.azure.com/
 - Make it public (probably)

Note that Azure associates only _one_ of your projects with a given
organization's GitHub Apps install, so you _cannot_ have multiple Azure
Projects that are linked to different GitHub projects under the same
GitHub user/organization. This is stupid, but such is life.

This template uses [Build
stages](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/stages),
which is a [preview
feature](https://docs.microsoft.com/en-us/azure/devops/project/navigation/preview-features)
of Azure Pipelines. You therefore need to enable support for it. To do
so, click your profile icon in the top-right corner and click "Preview
features". In the resulting box, enable "Multi-stage pipelines".

Now there's another fun step you have to do. Go to "Project settings"
(bottom left), the "Service connections" (under "Pipelines"). There
should be one thing listed there. Note down its name.

Now, create a file `azure-pipelines.yml` in the root of the repository
you want CI for. If you want all the bells and whistles, write:

```yaml
stages:
 - template: azure/stages.yml@templates

resources:
  repositories:
    - repository: templates
      type: github
      name: jonhoo/rusty-pipes
      endpoint: PLACEHOLDER
```

Where `PLACEHOLDER` is the service connection name we found above.

Once that's all done, it's time to set up the Pipeline in Azure:

 - Go to https://dev.azure.com/
 - Click the appropriate project
 - Click "Pipelines"
 - Click "New Pipeline"
 - Click "Use the classic editor to create a pipeline without YAML.".
   You must do this. If you just click GitHub, you're taken to the OAuth
   authentication page that surrenders all your secrets.
 - Click "GitHub"
 - Choose your repository using the triple-dot button
 - Click "Continue"
 - Give your pipeline a name -- probably the name of your project
 - Click "Apply" next to "YAML"
 - Under "Agent pool", select "Hosted"
 - Under "YAML file path", select "azure-pipelines.yml"
 - Click "Save & queue" towards the top.
   And then click it again...
 - Click "Save and run" bottom right

*Hopefully* Azure was now happy with your efforts. If it is, you'll be
taken to your new shiny "Pipeline summary" page, and it will show you
your build and tests progress! Congrats, you now have Azure Pipelines
CI! If you instead get a big red box at the top of the "Run pipeline"
box with an error, try to see if you can figure out which part of the
magic incantation you missed. If it all looks right to you, file an
issue!

## Showing off your new CI

If you want to add a status badge, click "Pipelines" on the left,
then your new pipeline, then the vertical tripe dots top right, the
"Status badge". While you're at it, add the badge to your `Cargo.toml`
too:

```toml
[badges]
azure-devops = { project = "AZURE_USER/AZURE_PROJECT", pipeline = "PIPELINE_NAME", build = "FOOBAR" }
```

Where `FOOBAR` is a weird extra parameter determined entirely by your
pipeline. When you have your pipeline open, look for `definitionId` in
the URL, and put the number you see there as `build`. If you don't do
this, your shown status badge will be correct, but it will link to the
wrong pipeline for… reasons.
