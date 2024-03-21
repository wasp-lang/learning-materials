# learning-materials

### Useful gists

- [Using Wasp 0.12 with ShadCN](https://gist.github.com/infomiho/b35e9366e16913949e13eaba0538f553)
- [Using Wasp with @fontsource (Solution for the "The request url * is outside of Vite serving allow list" issue)](https://gist.github.com/infomiho/9682e664948b84112074a69268f5673a)
- [Using Wasp with RadixUI](https://gist.github.com/infomiho/a18421740c205d1794c36c274bf09fc8)
- [Uploading files with Wasp 0.12+](https://gist.github.com/infomiho/ec379df4e33f3ae3410a251ba3aa81af)
- [Deploying Wasp with Caprover](https://gist.github.com/infomiho/6505d5970f5c334f704d658e9aa0bf56)
- [Using namespaces with Websockets in Wasp](https://gist.github.com/infomiho/14cf8b5b6efb07ba4f7a3e1ec76f4381)

### Wasp tutorials

You'll find many Wasp tutorials here: https://dev.to/wasp

## Other useful tips

### Setting meta tags for your landing page
- Having metatags set up is good for SEO and enables you set a cool preview picture when your app is shared in e.g. Slack, Twitter or Discord.
- If you are looking on way to set the metatags for your landing page, check out how we do it in the Open SaaS repo: https://github.com/wasp-lang/open-saas/blob/main/app/main.wasp#L7
- Note: this is only for the "main page", support for setting metatags for other pages is planned: https://github.com/wasp-lang/wasp/issues/911#issuecomment-2008111015

### Running `wasp db studio` on production DB

Or more generally, how to connect with Wasp to a production DB on Fly.

What's needed:
1. You'll need to get the database name and database password.
2. Then you'll need to open a tunnel.
3. Then you'll be able to execute DB command like `wasp db studio`.

Let's say our DB app name is `some-test-db`.

**Getting the DB name:**
```
fly postgres connect -a some-test-db
```
and then write the following:
```
\l
```
you'll find your DB name there.

By convention, it should be  `server_name_with_underscores`, for me it was `some_test_server`.

**Getting the DB password:**
```bash
fly ssh console -a some-test-db
```
then 
```
echo $OPERATOR_PASSWORD
```

With all this info, we can finally connect to the production DB:

**:one: Setting the database URL in `.env.server`**
Edit your `.env.server` to contain the following line (with correct values in the place of `<password>` and `<db_name>`):
```
DATABASE_URL=postgres://postgres:<password>@localhost:5432/<db_name>
```
e.g.
```
DATABASE_URL=postgres://postgres:myDatabasePassword@localhost:5432/some_test_server
```
**:two: Opening a tunnel so the production DB is available locally**
First, make sure nothing else is running on the port 5432, e.g. your local dev database (which you may have started with `wasp db start`, or in some other way). NOTE: even if you kill the terminal that was running your `wasp db start`, its Docker container still might be running in the background and occupying the port 5432 -> ensure that is not happening and terminate that container if needed.

Then, run
```bash
fly proxy 5432 -a some-test-db
```
Leave this terminal tab running and open a new terminal tab for future commands.
