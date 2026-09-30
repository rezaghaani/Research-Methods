# Route maps with live class sync

Two teaching pages (`session2.html`, `session3.html`) that students can follow on their own phones or laptops. When the lecturer moves to the next stop, every screen moves too. At a station the lecturer can open free time with a countdown; when the lecturer departs, every screen comes back. Class votes from phones appear as live bars on the lecturer's screen.

Without Firebase set up, the pages work as normal standalone files.

## What's in this folder

| File | Purpose |
|---|---|
| `index.html` | Landing page linking both sessions |
| `session2.html` | Sheet 1: finding and comparing research papers |
| `session3.html` | Sheet 2: AI land |
| `firebase-config.js` | Your Firebase settings (you paste these in, step 3) |
| `README.md` | This guide |

## One-time setup (about 15 minutes)

### 1. Put the pages on GitHub Pages

1. Sign in at github.com and create a new repository, for example `research-routes`. It must be **Public** for free GitHub Pages.
2. On the repository page, choose **Add file → Upload files**, drag in all the files from this folder, then **Commit changes**.
3. Go to **Settings → Pages**. Under **Build and deployment**, set **Source** to **Deploy from a branch**, choose **main** and **/ (root)**, then **Save**.
4. After a minute or two the address appears at the top of that page, like `https://YOUR-USERNAME.github.io/research-routes/`. Open it to check.

### 2. Create the Firebase project

1. Go to console.firebase.google.com and sign in with the Google account you'll present with.
2. **Create a project**. Any name works. Google Analytics isn't needed; you can switch it off.
3. In the left menu, open **Build → Realtime Database → Create Database**. Choose the location **Belgium (europe-west1)**, then **Start in locked mode**.
4. Open the **Rules** tab of the database, replace everything with the rules below, **put your own Google email address wherever it says `YOUR_GOOGLE_EMAIL`**, and click **Publish**.

```json
{
  "rules": {
    "sessions": {
      "$session": {
        ".read": true,
        "live":      { ".write": "auth != null && auth.token.email === 'YOUR_GOOGLE_EMAIL' && auth.token.email_verified === true" },
        "reveal":    { ".write": "auth != null && auth.token.email === 'YOUR_GOOGLE_EMAIL' && auth.token.email_verified === true" },
        "presenter": { ".write": "auth != null && auth.token.email === 'YOUR_GOOGLE_EMAIL' && auth.token.email_verified === true" },
        "call":      { ".write": "auth != null && auth.token.email === 'YOUR_GOOGLE_EMAIL' && auth.token.email_verified === true" },
        "assign":    { ".write": "auth != null && auth.token.email === 'YOUR_GOOGLE_EMAIL' && auth.token.email_verified === true" },
        "reports": {
          ".write": "auth != null && auth.token.email === 'YOUR_GOOGLE_EMAIL' && auth.token.email_verified === true",
          "$stop": { "$uid": { ".write": "auth != null && auth.uid === $uid", ".validate": "newData.hasChildren(['p','v']) && newData.child('p').isNumber() && (newData.child('v').isNumber() || (newData.child('v').isString() && newData.child('v').val().length < 200))" } }
        },
        "votes": {
          ".write": "auth != null && auth.token.email === 'YOUR_GOOGLE_EMAIL' && auth.token.email_verified === true",
          "$stop": { "$uid": { ".write": "auth != null && auth.uid === $uid", ".validate": "newData.isString() && newData.val().length <= 2" } }
        },
        "presence": { "$uid": { ".write": "auth != null && auth.uid === $uid", ".validate": "newData.isBoolean() || newData.isNumber()" } },
        "passengers": {
          ".write": "auth != null && auth.token.email === 'YOUR_GOOGLE_EMAIL' && auth.token.email_verified === true",
          "counter": { ".write": "auth != null", ".validate": "newData.isNumber() && newData.val() === (data.exists() ? data.val() + 1 : 1)" },
          "byUid": { "$uid": { ".write": "auth != null && auth.uid === $uid && !data.exists()", ".validate": "newData.isNumber()" } }
        },
        "callResp": {
          ".write": "auth != null && auth.token.email === 'YOUR_GOOGLE_EMAIL' && auth.token.email_verified === true",
          "$uid": {
            ".write": "auth != null && auth.uid === $uid",
            "link":   { ".validate": "newData.isString() && newData.val().length < 500 && newData.val().beginsWith('https://')" },
            "closed": { ".validate": "newData.isBoolean()" },
            "$other": { ".validate": false }
          }
        }
      }
    }
  }
}
```

Replace every `YOUR_GOOGLE_EMAIL` (there are nine). Only that account can move the class, open stations or reveal answers. Students can only follow, and write their own anonymous vote.

5. In the left menu, open **Build → Authentication → Get started**. On the **Sign-in method** tab:
   - enable **Anonymous** (students join without an account), and
   - enable **Google** (for you as presenter; pick your email as the support email).
6. Still in Authentication, open **Settings → Authorized domains → Add domain** and add `YOUR-USERNAME.github.io`.

### 3. Connect the pages to Firebase

1. In Firebase, click the gear icon → **Project settings**. Under **Your apps**, click the web icon **`</>`**, give it a nickname, and **Register app**. You don't need Firebase Hosting.
2. Firebase shows a `firebaseConfig` block. Copy its values.
3. On GitHub, open `firebase-config.js`, click the pencil icon to edit, and replace the `PASTE_...` values with yours. Check that `databaseURL` is filled in: it looks like `https://your-project-default-rtdb.europe-west1.firebasedatabase.app`. If it's missing, copy it from the top of the Realtime Database page. **Commit changes**.

Firebase's menu names change occasionally; if a label differs slightly, look for the nearest match.

### 4. Test with your phone (5 minutes, before the first class)

1. On your laptop open `https://YOUR-USERNAME.github.io/research-routes/session3.html#present` (note the `#present` at the end).
2. Click **Sign in with Google**. The QR code appears. Click **Start live session** in the bottom-left bar.
3. Scan the QR code with your phone. The phone shows **Travelling with the lecturer**.
4. Press → on the laptop: the phone follows. Go to a **TRY IT** stop (for example stop 7 in Session 3), set the minutes and press **Start hands-on**. The task sheet and countdown appear on the phone. Then press **Depart now**.
5. Press **Call → Random**: the phone shows the question mark. Paste any https link on the phone and check **Open link** appears on the laptop, then press **Done**.
6. At a class vote, vote on the phone, see the bars on the laptop, then click the right answer on the laptop to reveal it on the phone.
7. Click **⋯ → End live** when done.

## In class

- **Start:** open `session3.html#present` (or `session2.html#present`) on the classroom PC, sign in, click **Start live session**, and leave the QR code up while students join. Press **QR** in the bottom bar to show it again.
- **Talk and move on:** press → (or click a station). Everyone travels with you. There is no timer at ordinary stops.
- **Hands-on stations:** stops marked **TRY IT** on the map have a task for students. When you arrive at one, the bottom bar shows **− 10′ +** and **Start hands-on**. Set the minutes with − and +, then press **Start hands-on**. Every student gets a task sheet over their page (what to do, a prompt to copy, what to record) with a countdown, and can open any station while the time runs. Your projector shows the same sheet. **+5′** adds time. **Depart now**, or simply pressing →, brings everyone back and closes the sheet.
- **Platform stations (helping Niamh):** three hands-on stops split the class across platforms. When you press **Start hands-on**, each passenger is given one platform at random, balanced so each gets about the same number. Everyone works alone on Niamh's question and sends back one finding. Press **Findings** to show the pooled results: a dot per passenger and the median for each platform (for counts), or a table of DOIs showing which were found on several platforms and which on only one. Findings are anonymous and are deleted by **⋯ → Reset votes and findings** or the next **Start live session**.

  | Session | Stop | Platforms | Each passenger sends |
  |---|---|---|---|
  | 2 | The query | Scopus · Web of Science · Google Scholar · Google | Relevant papers in the first 20 results |
  | 2 | Backward and forward | Scopus · Google Scholar · Semantic Scholar | DOI of the most relevant paper citing Etxeberria et al. (2007) |
  | 3 | Discovery prompt | Elicit · Semantic Scholar · Gemini · Google Scholar (no AI) | DOI of the best candidate, checked |

  Change the platforms to match what your library and students can access: edit the `platforms` list for that stop in `TASKS`.
- **Free time without a task:** **⋯ → Free time 5′ or 10′** opens any stop for exploration without a task sheet.
- **Passengers:** each student gets a passenger number when they join (no names). It shows at the top of their screen. Numbers start again from 1 each time you press **Start live session**.
- **Calling someone:** press **Call** in the bottom bar, type a number or press **Random**. **Random** plays a visible draw on every screen: numbers of the connected passengers spin and land on one (never the one you just called), so the class sees it's chance. A typed number is called straight away. Their screen shows a large question mark. The projector shows "Passenger 4". If they want, they can paste a link to their own slides or document; it appears on your screen with **Open link**. The call ends when you press **Done**; they can also close their question mark themselves. Nothing is kept: the link is deleted when the call ends.
- **Votes:** students tap an option on their phones. Bars appear on your screen. Click the correct option on your screen to reveal the answer and the results on every phone. **⋯ → Reset votes** clears them.
- **Finish:** **⋯ → End live**. The page then works as a normal, free-to-explore map for everyone, including students revisiting it later.

### Hands-on stations and their default times

| Session | Stop | Task | Default |
|---|---|---|---|
| 2 | The query | Build and run your own query | 10 min |
| 2 | Screening | Screen five records | 8 min |
| 2 | Backward and forward | Snowball from one seed | 8 min |
| 2 | ResearchRabbit | Explore with ResearchRabbit (free account needed) | 10 min |
| 2 | Comparison matrix | Start your comparison matrix | 10 min |
| 3 | Discovery prompt | Run a discovery search | 10 min |
| 3 | Verify a reference | Verify one reference | 5 min |
| 3 | A notebook of her papers | Ask your own paper | 10 min |
| 3 | Extraction prompt | Extract and audit one row | 10 min |

The default is only a starting point; set the actual minutes in class with − and +. Outside a live session, each of these pages has a **Try it** button that opens the same task sheet, so students can do the tasks later on their own.

To add or change a task, edit the `TASKS` list near the top of the script in `session2.html` or `session3.html`: each entry has the stop id, default minutes, title, steps, an optional prompt, and what to record.

## Good to know

- **Capacity:** Firebase's free Spark plan allows up to 100 simultaneous connections. Each student's open page is one connection, so a class of 60 fits.
- **Privacy:** students join anonymously as passenger numbers. No names or emails are collected. Votes are stored under random IDs until you reset them, and a shared link only exists until you end that call.
- **Public pages:** anyone with the address can view the pages. Only your Google account can control the class.
- **If Wi-Fi or Firebase fails:** each page still works as a normal standalone page, so you can carry on presenting from the classroom PC.
- **Phones:** the pages switch to a one-column layout on phones. For the live-demo stops where students try NotebookLM or Elicit themselves, laptops work better.
