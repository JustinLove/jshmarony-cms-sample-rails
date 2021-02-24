# jsHarmony-CMS/Rails Integration sample

## Using this example

### Rails

    cd rails_jsh
    bundle install
    rails s

### jsharmony-cms-sample

Follow instructions at [https://github.com/apHarmony/jsharmony-cms-sample]

Log in and go to Sites.

Select `+ Add Site`

Enter a name, make active, and `Save`

Under `Deployment Targets` select `+ Add Deployment Target`

- Enter a name such as `01 / Rails Public`
- active status.
- Enter the full url path where you want to publish files, e.g. `file://c:/wk/jsharmony-cms-sample-rails/rails_jsh/public/cms/`
  - **IMPORTANT**: Do not use the `public` directory. The CMS manages all files below here, including deleteing files.
- params is a json object of values that will be made available to template publishing. The key field is the base url.

    {
      "env": "local",
      "content_url": "/cms/",
      "editor_base_url": "http://localhost:3000"
    }

Select `Save`

Select `Permissions`. Choose `All Users` for both edit and publish. Select `Save`, and close the deployment targets window.

Change to `Page Templates` tab and `+ Add` with

`Basic Page`, `Basic Page`, `REMOTE`, `%%%editor_base_url%%%/cms_templates/basic_page.html`

Hit `Save` and change back to the `Overview` tab.

- Set `Default Page Template` to `Basic Page` 
- Set `Default Preview/Editor` to `01 / Rails Public`

Hit `Save` and close the site window.

Select `Branches` on the top menu, and `+ New Empty Branch`

- select you site name
- choose `public`
- enter a name for the main branch. This is often called `master`, but names are global and this is already used by the default site. Perhaps `rails-master`.

jsHarmonyCMS supports a branch, review, merge, publish lifecycle, but we will be editing and publishing master here for brevity.

Saving the new branch will cause you to checkout the new branch (you have exactly one active branch at any time) and change to the `Pages` tab.

You will actually be on the `Sitemap` subtab.  Choose `+ Add`, create new primary sitemap, and `Save`.

This will put you into sitemap edit mode. (You may choose `Sitemap Confing` to go back to the previous screen.)

Right-click the `(Root)` item to add a child page, such as `About`.

Choosing `Edit Page Content` Should show the page in the rails application template. Click on the title or body to edit, and `Save` when happy.

You can build out the content as much as you like, but one page is sufficient to demonstrate the publishing and integration process.

Got to `Publish` on the top menu, `Publish` subtab, and select `+ Schedule Deployment`

- Site: Rails Sample
- Deployment Target: 01 / Rails Public
- Branch: rails-master
- hit `Schedule Deployment`

You can select `View log` to monitor the progress

Your rails project should now have a `public/cms/about/index.html`, and it should be directly viewable at [http://localhost:3000/cms/about/index.html]

We need one more step to have the cms control paths.

Go back to top level `Sites` tab, and edit your site.

Select the `Components` tab.

`+ Add` a new REMOTE template. Call it something like `rails_redirects`. For URL, enter `%%%editor_base_url%%%/cms_components/redirects.html` and `Save`

Close the site window and publish the site again. If configured correctly, you should see `public\cms\config\redirects.json` in the publish log. Once complete you should be able to go to [http://localhost:3000/about/] (without the /cms/)

You can also manage arbitrary redirects under the `Pages`, `Redirects` menu. (Every change needs a publish to be visible on the rails site)

### jsHarmony

    cd jsharmony_rails
    yarn install
    node node_modules/jsharmony-factory/create.js

Take note of initial login name and password.

Start the server

`./nstart.cmd`

There will be many warnings, but we need to run the web interface to run the CMS setup scripts to fix them.
Log in to the web interface (default [http://localhost:8080]) with the credentials from the create step.
Navigate to Developer -> DB Script
Under jsHarmonyCMS, run
  - `init`
  - `restructure`
  - `init_data`
  - `sample_data`

You should now be able to quit the server and restart without errors.

Navigate to Sites, select `Manage Site`. For `01 / Local FS` Select `Edit`

  - change the Path setting to reflect the absolute path on your local filesystem.
  - Check the params to ensure the address of the rails server is correct for how you are running it.

Change to `Page Templates` tab and `+ Add` with

`Basic Page`, `Basic Page`, `REMOTE`, `%%%base_url%%%/cms_templates/basic_page.html`

Hit `Save`

Select `Pages` on the top menu. On both `Home` and `Style Guide`,

  - Select `Page Properties`
  - Change Template to `Basic Page`
  - Save properties

## Outline of How This Was Made

### Rails setup

(install rails and prereqs as needed)

`rails new rails_jsh`

Added a token random_numbers controller and view.

Added a cms_templates controller, and a basic_page view, with wildcard routing

`get "/cms_templates/*template", to: "cms_templates#index"`

See the view file for required contents. There is also a cms_templates_helper that has the cms javascript tag (which includes the url of the cms managment server)

Added a cms_components controller, and a redirects.html view, with wildcard routing. This controller is nealry same as templates, but components have no layout applied.

`get "/cms_components/*template", to: "cms_components#index"`

Notice that `app/views/cms_components/redirects.html` is *not* an `.erb` file. The template tags here are interpreted as EJS by the CMS. If you also want to perform ERB processing, you will need to take care to escape any EJS tags.

The redirects component allows the CMS to publish a `redirects.json` file, which is loaded by `/lib/middleware/cms_router_middleware.rb`. In this sample, the redirect file is loaded on every request for simplicity.

`application.rb` loads the `CmsRouterMiddleware`. It is placed early in the chain so that it can redirect to static files and have the standard static file middleware take care of it.

### jsharmony-cms-sample setup

Follow instructions at [https://github.com/apHarmony/jsharmony-cms-sample]

### jsHarmony-CMS setup

(install jsharmony-cli from npm as needed)

`jsharmony create factory`

- SQLite
- No Client Portal

`yarn add jsharmony-cms`

Edit `app.js`, replacing instances of factory with cms.

`yarn add jsharmony-image-sharp`

Edit `app.config.js` adding:

`  jsh.Extensions.image = require('jsharmony-image-sharp');`

Start the server

`./nstart.cmd`

There will be many warnings, but we need to run the web interface to run the CMS setup scripts to fix them.
Log in to the web interface (default [http://localhost:8080]) with the credentials from the create step.
Navigate to Developer -> DB Script
Under jsHarmonyCMS, run
  - `init`
  - `restructure`
  - `init_data`
  - `sample_data`

You should now be able to quit the server and restart without errors.

Add models/sql/objects/deployment.json (see file)

Create the files

    /views/jsh_cms_editor.css.ext.ejs
    /views/jsh_cms_editor.ext.ejs

Since these are new files, restart the server if it was running.

Navigate to Sites, select `Manage Site`. For `01 / Local FS` Select `Edit`

Change to `Page Templates` tab and `+ Add` with

`Basic Page`, `Basic Page`, `REMOTE`, `%%%base_url%%%/cms_templates/basic_page.html`

Hit `Save`

Select `Pages` on the top menu. On both `Home` and `Style Guide`,

  - Select `Page Properties`
  - Change Template to `Basic Page`
  - Save properties


? Can template be defined a similar manner to default deployment