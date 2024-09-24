# learning-materials

### Useful gists

- [Using Wasp 0.12 with ShadCN](https://gist.github.com/infomiho/b35e9366e16913949e13eaba0538f553)
- [Using Wasp with @fontsource (Solution for the "The request url * is outside of Vite serving allow list" issue)](https://gist.github.com/infomiho/9682e664948b84112074a69268f5673a)
- [Using Wasp with RadixUI](https://gist.github.com/infomiho/a18421740c205d1794c36c274bf09fc8)
- [Uploading files with Wasp 0.12+](https://gist.github.com/infomiho/ec379df4e33f3ae3410a251ba3aa81af)
- [Deploying Wasp with Caprover](https://gist.github.com/infomiho/6505d5970f5c334f704d658e9aa0bf56)
- [Using namespaces with Websockets in Wasp](https://gist.github.com/infomiho/14cf8b5b6efb07ba4f7a3e1ec76f4381)
- [Implementing a custom OAuth provider (Spotify)](https://gist.github.com/infomiho/3c63de7d53aba59d6293bcb59501a029)
- [Multiple domains support for CORS using custom global middleware](https://gist.github.com/infomiho/5ca98e5e2161df4ea78f76fc858d3ca2)
- [Deploy Wasp to a VPS (reverse proxy + Docker)](https://gist.github.com/infomiho/80f3f50346566e39db56c5e57fefa1fe)

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

### Testing Wasp apps on local network

Sometimes when you develop locally, you'll want to check your app on some other device. For example, this is useful for seeing if your app performs well on phones and tablets.

Here's how you can do it with a Wasp app:
1. Simply run your app with `wasp start`
2. Look for this part of Wasp's output in the terminal:
    ```diff
     [ Client ]   VITE v5.2.6  ready in 229 ms
     [ Client ]
     [ Client ]   ➜  Local:   http://localhost:3000/
    +[ Client ]   ➜  Network: http://192.168.1.39:3000/
    +[ Client ]   ➜  Network: http://198.19.249.3:3000/
    +[ Client ]   ➜  Network: http://192.168.215.0:3000/
     [ Client ]   ➜  press h + enter to show help
    ```
3. Try opening the app on your phone using one of the URLs.

    You might get just one **Network** URL, use that one. 
    If you got multiple network interfaces on your machine, you'll get more URLs, just try them all until you see the client of your app open.

6. **Important:** the app is still not functional. For that we need to adjust some env variables.
   
    Edit `.env.server` to contain:
   ```env
   WASP_WEB_CLIENT_URL=http://192.168.1.39.nip.io:3000
   #                   ^ This is the URL with the IP you picked in step 3 above
   
   # You'll notice we wrote the URLs like this:
   #
   # http://<your-ip>.nip.io:<port>
   #
   # 1. nip.io is a simple DNS service that we can use during development (we'll explain below a bit more)
   # 2. The <port> part is 3000 for the client, and 3001 for the server
   WASP_SERVER_URL=http://192.168.1.39.nip.io:3001
   ```
   And the `.env.client` to contain:
   ```env
   REACT_APP_API_URL=http://192.168.1.39.nip.io:3001
   ```

    <details>
      <summary>Why we defined those env vars</summary>
  
      - We defined `WASP_WEB_CLIENT_URL` to make sure CORS works.
      - We defined `WASP_SERVER_URL` to make sure OAuth redirects work. You'll need to adjust your redirect URLs for each of the OAuth providers as well.
      - We defined `REACT_APP_API_URL` so our client where to find our server on the local network, otherwise it won't work.
    </details>
     
7. Try loading your website again on your phone, for the example above, we would now open `http://192.168.1.39.nip.io:3000`

That should be it!

**Why use used nip.io**: for some parts of Wasp (for example Google Auth) simple local IPs aren't allowed. You can skip using `nip.io` if you want, of course, but this is just being extra careful since there is not downside to using `nip.io`. It's free and it just works.
