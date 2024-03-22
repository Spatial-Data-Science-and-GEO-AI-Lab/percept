# Setting up the frontend server

## Getting the code

You should have NodeJS installed as described in the Dependencies section of the [tutorial start](README.md). Obtain a copy of the `percept-frontend` source code from GitHub and put it into any convenient directory. For example, `cd $HOME && git clone https://github.com/Spatial-Data-Science-and-GEO-AI-Lab/percept-frontend && cd percept-frontend`.

## Configuration

Copy `src/config.js.sample` to `src/config.js` and edit the values in the file according to the comments.

The contents of the configuration file look like this:

    // URL root of the backend server, with the full domain name
    const backendURL = 'http://<domain_name>/backend/api/v1';

    // After a rating button is pushed, this is the number of milliseconds to wait
    // before allowing another button push. To prevent accidental clicks.
    const buttonReenableTimeout = 1000;

    // The person responsible for managing GDPR related issues.
    const gdprControllerName = "My Name";

    // The contact e-mail for said GDPR manager.
    const gdprControllerEmail = "My E-mail Address";

    // END OF CONFIGURATION

    // Export configuration:
    module.exports = { backendURL, buttonReenableTimeout, gdprControllerName, gdprControllerEmail }

## Installing the software

In the project directory, run: `npm install`

This should downloand and install all the NodeJS dependencies.

## Configuring Apache2

Recall that when you [configured the image server](apache.md) you placed a `ProxyPass` directive inside the `VirtualHost` block, and you also did something similar for the [backend](backend.md). Now it is time to continue adding additional `ProxyPass` directives, for the frontend. Simply copy these lines into your Apache2 server configuration file immediately after the existing `ProxyPass` directives:

    ProxyPassMatch          "^/static/(css|js)/(.*)(js|css|map|txt)$" http://localhost:3000/static/$1/$2$3
    ProxyPassMatch          "^/(report|eval)$" http://localhost:3000/$1
    ProxyPassMatch          "^/(rate_sample.\.jpg)$" http://localhost:3000/$1
    ProxyPassMatch          "^/(mapillary_logo\.png)$" http://localhost:3000/$1
    ProxyPassMatch          "^/(favicon\.ico)$" http://localhost:3000/$1
    ProxyPassMatch          "^/(manifest\.json)$" http://localhost:3000/$1
    ProxyPassMatch          "^/(asset-manifest\.json)$" http://localhost:3000/$1
    ProxyPassMatch          "^/(robots\.txt)$" http://localhost:3000/$1
    ProxyPassMatch          "^/(logo...\.png)$" http://localhost:3000/$1
    ProxyPassMatch          "^/(index\.html)$" http://localhost:3000/$1

Reload your Apache configuration.

## Running the frontend server

Within the `percept-frontend` directory you have two options for running the software. First, for testing and development, you can use the command:

    npm run dev

This will run a development and debugging server that you can use to try out the app. Later on, when you are ready to deploy for production, you can instead run:

    npm run build && serve -s build

This compiles a production version of the frontend app and begins serving it.

Either way, we suggest running this inside of a detachable terminal using a program like `screen`, `tmux`, `byobu` or at the very least `nohup` so that it keeps running even if you close the terminal or disconnect your session from the server.

## Testing the frontend

You should now be able to navigate your web browser to `https://<domain_name>/` to see if it works.
