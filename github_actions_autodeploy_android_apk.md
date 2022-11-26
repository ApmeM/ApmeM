## Deploy android application using github actions.

### Create service account

A service account is an entity that you create that tells services or applications it interacts with that it is operating on your behalf.

In our case, our GitHub Action is going to interact with the Google Play Store so it can upload a new version of our application.

To create a service account, go to the <a href="http://console.cloud.google.com">Google Cloud Console</a> If you have no account there, make sure to create one. 

- Go to "IAM and admin > Service Accounts".
- Click "+Create service account".
- On the first step set any name.
- On the second step in the "Role" section select "Basic > Editor".
- On the third step set your email under "Grant users access to this service account".


Now we need to create a key for the newly created account.

- Open newly created account
- Open keys tab
- Select "New Key > Create new key > JSON". 
- Json file will be automatically downloaded.

Now we need to make Google Play aware of this service account. Go to <a href="https://play.google.com/console">Google Play Console</a>

- Go to "Settings > API Access<.
- Find section "Service Accounts". You should be able to see the service account you created previously. Click the Grant Access link
- In the settings that open, head over to App permissions. Here you will choose which application this service account interacts with. 
- Under Account permissions, everything under the "releases" section should be checked. 
- Click the "Invite user".
- After the invitation is sent, we can run the publishing to store action.

### Usage example

<code>
  
      - name: Deploy to Play Store
        uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJsonPlainText: ${{secrets.SERVICE_ACCOUNT_KEY}}
          packageName: com.tomerpacific.laundry
          releaseFiles: path/to/release.apk
          track: production
</code>

### Known errors

#### Changes cannot be sent for review automatically. Please set the query parameter changesNotSentForReview to true. Once committed, the changes in this edit can be sent for review from the Google Play Console UI.

This error means you have unsubmitted changes in play console (different policies or accept rules).
Submit them (some of them should be reviewed) and after everything is done this error will disappear.
