# Runtime Shopify Theme
This is the repository for the Runtime Shopify Theme files and data.

### Install Theme Kit
Theme Kit is a command line tool maintained by Shopify to allow developers to download, update, and deploy theme files for their stores. We will use Theme Kit to help ensure that the live and version controlled theme files are synchronized whenever changes are necessary.

For macOS installation use Homebrew:
```
brew tap shopify/shopify
brew install themekit
```
[Instructions]([https://shopify.github.io/themekit/#installation](https://shopify.github.io/themekit/#installation)) for Windows, Linux and other operation systems

### Initial setup
After installing Theme Kit the next step is to clone the theme repo and setup the `.env` file. The `.env` file should be located at the root level of the repo and contains the api passwords for the Staging and Production stores. These can be found in the Shopify Admin panel under `Apps -> Manage Private Apps -> theme-kit` on both Production and Staging. **The `.env` file should never be committed to the repo to keep the api passwords secure.**

The `.env` file should be setup as such:
```
STAGING_PWD=[staging_api_password]
PRODUCTION_PWD=[production_api_password]
```

### Building new features
The Shopify platform presents several challenges to ensuring that code changes are controlled and reliable. This process attempts to minimize the chance that broken or undesirable changes will be deployed to any store or environment.
1. **Duplicate the existing Production and Staging themes**
	From the Shopify Admin panels for both Production and Staging navigate to `Online Store -> Themes -> [current theme] -> Actions -> Duplicate`. This will produce a new theme called `Copy of [current theme]`  
2. **Rename the new duplicate themes** from `Copy of [current theme]` to `[current theme]-[feature name]` providing a descriptive feature name
3. **Get the theme ids** for the Production and Staging feature themes
	Use Theme kit's `get` command to retrieve the theme id for each of the new duplicate themes:
	```
	theme get --list -p=[api-password] -s=[your-store.myshopify.com]
	```
	This will return something like:
	```
	[99904585892][live] [current theme]
	[99916382372] [current theme]-[feature name]
	```
	The theme id is the number in the first set of square brackets. Note these values to be used later.
4. **Switch to the local repo folder and make sure you are on the `master` branch.**
5. **Download a copy of the current Production theme**
	Shopify provides an interface for editing theme files and data and it is possible for the live Production theme to be out of sync with the `master` branch in this repo. This step ensures any conflicts are resolved by pulling differences from Production into the `master` branch. Use Theme Kit's `get` command to download the theme:
	```
	theme get -p=[spi-password] -s=[your-store.myshopify.com] -t=[theme-id]
	```
6. **Commit any differences to the `master` branch**
	Review any differences in the files and commit them to `master` with a commit message like `sync to production files`. Note: The `config.yml` file will always be changed as Theme Kit's `get` command overwrites the `development` environment data within the file. It's not necessary to commit just this file.
7. **Update the `config.yml` file** with the theme ids (and store information if necessary)
	The `config.yml` file will look something like this:
	```
	development:
	  password: ${STAGING_PWD}
	  theme_id: "99904585892"
	  store: your-store-dev.myshopify.com
	staging:
	  password: ${STAGING_PWD}
	  theme_id: "99904585892"
	  store: your-store-dev.myshopify.com
	production:
	  password: ${PRODUCTION_PWD}
	  theme_id: "99695394977"
	  store: your-store.myshopify.com
	```
	Replace the `theme_id` value for both Staging and Production with the correct theme ids from Step 3 and verify that the `store` values are correct, then save the file. The placeholder variables will be replaced with the values from the `.env` file. Ignore the `development` environment variables and values.
8. **Cut a new feature branch from master** using the feature name from Step 2
9. **Make necessary feature changes to theme files and data**.
10. **Deploy feature branch to the Staging feature theme**
	Once the feature changes are complete use the Theme Kit `deploy` command to copy the files into the Staging feature theme:
	```
	theme deploy --env=staging --vars=.env
	```
11. **Verify feature changes on the Staging store** by using either the Preview theme action in the Shopify Admin panel, found under `Online Store -> Themes -> [current theme]-[feature name] -> Actions -> Preview` or by publishing the feature theme to the Staging store using `Online Store -> Themes -> [current theme]-[feature name] -> Actions -> Publish`.  Certain features may not be testable in preview mode and any automated frontend tests with likely require the changes to be published live.
12. **Merge feature branch into `master`** once the changes have been approved and are ready to be deployed to the Production store.
13. **Deploy master branch to production feature theme** using Theme Kit's `deploy` command:
	```
	theme deploy --env=production --vars=.env
	```
14. **Preview feature changes in the Production store** using the Preview theme action. This is to give one final opportunity to verify the changes in Production before making them live.
15. **Rename the feature theme in Production**. Once the feature changes have been approved to go into production use the Rename theme action to change the feature theme from `[current theme]-[feature name]` to `production-v.x.x.x`, adding the correct version number.
16. **Publish the renamed theme in Production** using the Publish theme action
17. Git-tag the `master` branch with version number
18. Push the `master` branch changes **and tags** to GitHub.
