# CauseConnect 💙

**A simple, honest crowdfunding platform where people can start a cause, back a cause, or keep the whole thing in check.**

Built for **PS No. 15 – Online Crowdfunding Platform** as part of the Java Programming project evaluation (GUVI × HCL × Galgotias University).

---

## So, what is this?

Imagine a teacher who wants to raise money for books for kids who can't afford them. She needs a place to post that goal. A neighbour wants to chip in ₹500. And someone has to make sure the campaign is real before money starts flowing.

That's CauseConnect. It's a small web app with three kinds of people:

| Who | What they can do |
|---|---|
| **Creator** | Start a campaign, track how much it has raised, post updates for supporters |
| **Contributor** | Browse campaigns, donate, and look back at everything they've supported |
| **Admin** | Review new campaigns, approve or reject them, and keep an eye on users and contributions |

One rule we're quietly proud of: **you can only donate to a campaign an admin has approved.** If someone tries to donate to a rejected one, the system says no.

---

## How it's put together

We didn't want a pile of frameworks we couldn't explain in a viva, so we kept it plain on purpose.

```
Browser (HTML / CSS / a bit of JavaScript)
        │   HTTP + JSON
        ▼
Java backend  (built-in com.sun.net.httpserver)
        │
        ▼
DAO layer  →  UserDAO · CampaignDAO · ContributionDAO · CampaignUpdateDAO
        │
        ▼
JDBC
        │
        ▼
MySQL  →  users · campaigns · contributions · campaign_updates
```

A quick honest note: early on we planned to use Servlets/JSP. In the end we went with Java's own lightweight `HttpServer` and sent JSON back and forth with Gson. It kept things simpler, and the Java, JDBC and MySQL core is exactly what we originally set out to build.

**Tech we used**

- Java (core OOP, collections, threads for the server's request pool)
- JDBC with `PreparedStatement`, including transactions for contributions
- MySQL
- Gson for JSON
- Plain HTML, CSS and JavaScript on the front

---

## What's inside the folder

```
causeconnect/
├── Backend/
│   ├── lib/                 # gson + MySQL connector jars
│   ├── src/
│   │   ├── model/           # User, Campaign, Contribution, CampaignUpdate
│   │   ├── dao/             # All the database talking happens here
│   │   ├── database/        # DBConnection (one shared place for the connection)
│   │   ├── service/         # Login logic
│   │   └── server/          # CauseConnectServer: API routes + serves the frontend
│   └── out/                 # Compiled classes
└── Frontend/
    ├── *.html               # Login, register, dashboards, campaigns, admin pages...
    ├── css/style.css
    ├── js/                  # api.js, auth.js, main.js
    ├── assets/images/       # Logo
    └── CROWDFUNDING.sql     # Database script
```

---

## Want to run it? Here's how

### 1. What you need first

- **JDK 11 or newer** (we developed on a recent JDK)
- **MySQL Server** running on your machine
- A terminal. We used PowerShell on Windows, so the commands below are written for that.

### 2. Set up the database

Open MySQL (Workbench or the command line) and run the script in `Frontend/CROWDFUNDING.sql`. It creates the tables and adds three demo users.

> **Heads up:** the app connects to a database called `crowdfunding` (lowercase). On Windows MySQL doesn't care about capitals, but on Linux/macOS it does. If you're on one of those, create the database as `crowdfunding` in lowercase.

### 3. Tell the app how to reach MySQL

By default it tries:

- URL: `jdbc:mysql://localhost:3306/crowdfunding`
- User: `root`
- Password: `root`

If your MySQL password is different, set these before starting (no code changes needed):

```powershell
$env:DB_URL      = "jdbc:mysql://localhost:3306/crowdfunding"
$env:DB_USER     = "root"
$env:DB_PASSWORD = "your_password_here"
```

### 4. Compile

Run this from the **project root** (the `causeconnect` folder, not inside `Backend`):

```powershell
javac -cp "Backend/lib/*" -d Backend/out (Get-ChildItem -Recurse -Filter *.java Backend/src).FullName
```

### 5. Start the server

Still from the project root:

```powershell
java -cp "Backend/out;Backend/lib/*" server.CauseConnectServer
```

(On Linux/macOS use `:` instead of `;` between the paths.)

You should see a banner saying **CauseConnect Backend Server Running!** Now open:

**http://localhost:8080**

Want a different port? Just add it at the end, e.g. `... server.CauseConnectServer 9090`.

> **Why the project root matters:** the server looks for the `Frontend` folder relative to where you launch it. Start it from somewhere else and you'll get 404s. (We learned this one the annoying way. See the story below.)

---

## Try it out: demo logins

These come from the SQL script. All three share the same password.

| Role | Email | Password |
|---|---|---|
| Creator | `alice@test.com` | `1234` |
| Contributor | `bob@test.com` | `1234` |
| Admin | `admin@test.com` | `1234` |

A nice little walkthrough:

1. Log in as **Alice** and create a campaign. It starts out waiting for approval.
2. Log in as **Admin** and approve it.
3. Log in as **Bob**, find the campaign and contribute.
4. Go back to Alice's dashboard. The raised amount has gone up.
5. As Alice, post an update. Contributors can see it.

---

## The API, for the curious

The frontend talks to these endpoints:

| Route | Used for |
|---|---|
| `/api/auth` | Login and registration |
| `/api/campaigns` | Listing, creating and managing campaigns (and admin approval) |
| `/api/contributions` | Making and viewing contributions |
| `/api/updates` | Campaign updates from creators |
| `/api/users` | User info and admin user management |
| `/api/stats` | Totals for dashboards (campaigns, contributions, users) |

---

## The database in a nutshell

- **users**: `user_id`, `name`, `email` (unique), `password`, `role`
- **campaigns**: `campaign_id`, `creator_id`, `title`, `description`, `goal_amount`, `raised_amount` (starts at 0), `status`
- **contributions**: `contribution_id`, `campaign_id`, `user_id`, `amount`, `contribution_date`
- **campaign_updates**: `update_id`, `campaign_id`, `creator_id`, `title`, `message`, `update_date`

When someone contributes, we insert the contribution and bump the campaign's `raised_amount` inside **one JDBC transaction** (`setAutoCommit(false)` → commit). Either both happen or neither does, so the totals never drift or get counted twice.

---

## How we actually built it (the short, real version)

We didn't generate everything in one go. We built it layer by layer and tested each one before moving on:

1. Started as a **console app** with models and role-based menus, just to prove the logic.
2. Designed the **MySQL** schema.
3. Connected with **JDBC** and wrote the **DAO** classes.
4. Tested the pieces separately (connection, campaign insert/select, contribution select) so we always knew which layer was broken.
5. Added **login, admin approval/rejection, contributions and campaign updates**.
6. Moved to the **web version**: designed the UI (we used Google Stitch for design ideas), gave it the CauseConnect identity, and wired the frontend to the same backend.
7. Tested end to end.

**Things that bit us along the way**

- `No suitable driver found for jdbc:mysql...`: the MySQL Connector/J jar wasn't on the classpath. Putting it in `lib` and fixing the classpath solved it.
- We once ran commands from the wrong copy of the project and thought `Main.java` had vanished. It hadn't. We were just in the wrong folder.
- Stale `.class` files hid problems, so we got into the habit of clean-compiling.
- Contributions to **rejected** campaigns had to be blocked, so now they are.
- Making sure `raised_amount` was never counted twice.

You'll also notice a few odd campaigns in the database like "JDBC Test Campaign" or "Clean Code Test". Those aren't bugs. They're leftovers from all the testing.

---

## Where it falls short (being honest)

This is a student project, and we'd rather tell you than have you discover it:

- **Passwords are stored as plain text.** Fine for a demo, not okay for real use. Hashing (like BCrypt) is the first thing we'd add.
- **Sessions live in memory**, so restarting the server logs everyone out.
- **No real payment gateway.** Contributions are recorded, but no money actually moves.
- **CORS is wide open** (`*`) to make local development easy.
- The SQL script creates the `campaign_updates` table twice. If MySQL complains about the second one, you can safely ignore it. The table already exists.

---

## What we'd do next

- Hash passwords
- Real payment integration
- Email notifications for approvals and updates
- Campaign images and categories
- Search and filters
- Deadlines for campaigns

---

## The team

| Role | Name | Email |
|---|---|---|
| Team Leader | _your name here_ | _email_ |
| Member | _name_ | _email_ |
| Member | _name_ | _email_ |
| Member (if any) | _name_ | _email_ |

**Faculty mentor:** _name_
**Submission:** Java Programming Project, Review 1 (deadline 10 October 2026) and Review 2 (deadline 15 November 2026)

---

## A last word

CauseConnect isn't trying to be the next big fundraising site. It's a project where we got to take an idea from a blank folder to a working app: database, JDBC, DAOs, business rules, and a real interface, one bug at a time. If you run it and it just works, that's a good feeling. If it doesn't, check the project root, the MySQL password and the jar paths. That's where most of our problems came from too. 
