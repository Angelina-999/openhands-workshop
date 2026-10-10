# https://gcci.earth/

# Development

- Clone the repo
- Run `pnpm install`
- Run `pnpm run develop` to launch a local dev-server at http://localhost:8100/ with live reload

If you change translations in JSON-files, then you might need to repeat `pnpm run develop`.

# OpenHands

Our OpenHands instance runs at https://openhands.gcci.dev/.
Login works over whitelisted mail addresses — if yours is not whitelisted yet, ask for it to be added.

Issues can be submitted into OpenHands instead of being implemented by hand.
OpenHands then works on the issue and automatically creates a merge request containing:

- a link to the Netlify preview deployment of that branch,
- before/after screenshots for UI changes,
- the steps to verify the change.

For every new chat, select the workspace `gccivis` in the workspace selector instead of "No workspace" —
otherwise the agent starts without this repo.

<img src="doc/images/openhands-new-chat.jpg" width="300"> <img src="doc/images/openhands-workspace-dropdown.jpg" width="300">

The agent rules for this repo are in [CLAUDE.md](CLAUDE.md).

# URL Parameters

Available query parameters are documented in [doc/availableURLParameters.md](doc/availableURLParameters.md).

# Deployment

Nothing deploys to production automatically. Every deploy is a Netlify draft:

- pushes to `develop` are deployed as a draft by the GitLab pipeline,
- feature branches are deployed as a draft by OpenHands, linked from the merge request,
- promotion to https://gcci.earth/ happens manually in Netlify.

Deployments can also be triggered manually by running `pnpm run deploy`.
To preview deployments, log into Netlify and navigate to https://app.netlify.com/sites/gcci/deploys.
Click on "Publish deploy" to publish a specific deployment to https://gcci.earth/.

# JSON Updates

Updates of JSON-files should comply with the following procedure to ensure traceability and auditability:

- Replace selected JSONs within the subfolder `static`.
- Run `./scripts/prettify_json_assets.sh` to ensure that newly added JSONs have a standardized format with readable linebreaks.
- Optionally, edit `variable-list.js` if JSONs have been renamed or moved to a different sub-sub-folder.
- For small changes, make a Git-commit where the deletion of old JSONs and addition of new JSONs happens within the same commit. This makes it possible to see on GitLab what has changed in a specific commit.
- Push the Git-commit; review changes before deploying into production.

# Techstack

We use _React_ to build the UI and _D3.js_ to create the charts.

We use _Gatsby_ as a build-system to generate static sites.
Static site generation is helpful to optimize GCCI for search engines.
Therefore, we deliberately chose to _not_ build a single-page-application with toolchains like create-react-app.

A Gatsby Strapi boilerplate was used to bootstrap the stack. The stack includes:

- [Bootstrap](https://getbootstrap.com/)
- [Reactstrap](https://reactstrap.github.io/)
- [Reactstrap-form-fields](https://github.com/jeremylynch/reactstrap-form-fields)
- [Reactstrap-json-nav](https://github.com/jeremylynch/reactstrap-json-nav)
- [React Icons](https://github.com/react-icons/react-icons)
- [Google Tag Manager](https://github.com/gatsbyjs/gatsby/tree/master/packages/gatsby-plugin-google-tagmanager)
- [gatsby-plugin-netlify](https://github.com/gatsbyjs/gatsby/tree/master/packages/gatsby-plugin-netlify)
- [gatsby-plugin-google-tagmanager](https://www.gatsbyjs.org/packages/gatsby-plugin-google-tagmanager/)
- [gatsby-plugin-facebook-pixel](https://www.gatsbyjs.org/packages/gatsby-plugin-facebook-pixel)
- [gatsby-plugin-manifest](https://www.gatsbyjs.com/plugins/gatsby-plugin-manifest/)
- [gatsby-plugin-styled-components](https://www.gatsbyjs.org/packages/gatsby-plugin-styled-components/)

[View Gatsby Demo](https://gatsby-strapi-starter.netlify.com/)
