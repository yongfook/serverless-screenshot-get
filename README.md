# serverless-screenshot-get

Get PNG data of a fresh web page screenshot using AWS Lambda, PhantomJS and the [Serverless](https://github.com/serverless/serverless) framework.

NOTE: This is meant as a building block for a system which does caching and resizing. You probably do not want to serve these images directly on a web page.

# Example usage

To take a simple screenshot if you have deployed the lambda without a password:

    curl -s -G 'https://fav7ffggds.execute-api.us-east-1.amazonaws.com/dev/screenshot' \
        --data-urlencode 'url=http://phantomjs.org/' > screenshot.png

NOTE: `-G` just makes curl do a `GET` request with issued data as url query parameters.

There are also some other features available:

    curl -s -G 'https://fav7ffggds.execute-api.us-east-1.amazonaws.com/dev/screenshot' \
        --data-urlencode 'width=1280' \
        --data-urlencode 'height=1028' \
        --data-urlencode 'delayms=100' \
        --data-urlencode 'timeoutms=29000' \
        --data-urlencode 'clip=yes' \
        --data-urlencode 'clipwidth=1280' \
        --data-urlencode 'clipheigth=1024' \
        --data-urlencode 'cliptop=0' \
        --data-urlencode 'clipleft=0' \
        --data-urlencode 'clipwithiframe=yes' \
        --data-urlencode 'iframescrollto=0' \
        --data-urlencode 'iframescrolldelayms=50' \
        --data-urlencode 'evalcode=$("body").css("backgroundColor","black");' \
        --data-urlencode 'evaldelayms=10' \
        --data-urlencode 'preloadurl=http://phantomjs.org/login/' \
        --data-urlencode 'preloaddelayms=2000' \
        --data-urlencode 'preloadevalcode=$("#login").val("knight"); $("form").submit();' \
        --data-urlencode 'preloadevaldelayms=10' \
        --data-urlencode 'secret=myverysecret' \
        --data-urlencode 'url=http://phantomjs.org/' > screenshot.png

# Parameter descriptions

The only required parameter is `url`.

Secret can be configured to be required when deploying, but is not required by default.

Delay is time to wait before taking a screenshot after PhantomJS thinks the page is loaded.

Timeout is how long PhantomJS is given time to do everything, preloads and delays included.

Sometimes you want to enable `clipwithiframe` if you are dealing with pages that use scroll positions as navigation. Some sites (like google.com) do not allow you to browse through an iframe though, usually leading to a timeout, so it should not be used as default.

jQuery 1.12.4 is injected into the pages before any `eval` code is run.

If you preload a page where you log in using a JavaScript eval, you will need to increase `preloaddelayms` from the default 100 so that the browser has time to get and store your cookies.

# Installing

1. Create a new Serverless project using this repository
2. Generate AWS tokens for Serverless to deploying Lambda functions
3. Add a password and check the defaults in the Serverless environment
4. Deploy API to AWS Lambda
5. Add binary data support to API Gateway
6. Test your deployment

## Create a project

You need Node.JS and NPM installed. Newer serverless versions might work too, but this is what I have tested:

    $ npm install -g serverless@v1.16.1
    $ sls install -u https://github.com/amv/serverless-screenshot-get -n my-screenshot-get
    $ cd my-screenshot-get
    $ npm install

## Generate AWS token for deploying Lambda functions

Go to the Serverless.com [Quick start guide](https://serverless.com/framework/docs/providers/aws/guide/quick-start/) and set up your Access Key ID and Secret Access Key as instructed.

## Add a password and check the defaults in the Serverless environment

Optional: Edit your `serverless.yml` to add a password and to configure the default timeout. There is a `# NOTE` comment above the variables so that you can find them easier.

## Deploy API to AWS Lambda

    $ serverless deploy

Note the output of the last command, where you can get the URL for your API.

## Add binary data support to API Gateway

This step will hopefully go away in the future, but because of missing Cloud Formation features, Binary Support must be added by hand in the AWS Console:

 1. Open your region API Gateway console, for us-east-1: https://console.aws.amazon.com/apigateway/home?region=us-east-1
 2. Select the "dev-serverless-screenshot-get" API
 3. Choose "Binary Support"
 4. Add `*/*` and remember to press "Save".
 5. Choose "Resources"
 6. Pick "Deploy API" from the "Actions" dropdown.
 7. Select Deployment stage as "dev".
 8. Press "Deploy".

## Test your deployment

Here is an example to open in your browser. You should change the URL domain to match yours:

    https://fav7ffggds.execute-api.us-east-1.amazonaws.com/dev/screenshot?url=https://google.com/

# Acknowledgements

Most of the code is adapted from a [similar but more complex project](https://github.com/svdgraaf/serverless-screenshot) by Sander van de Graaf.
