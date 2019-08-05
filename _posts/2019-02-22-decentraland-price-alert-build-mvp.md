---
layout: post
title: "How I built an MVP in a Day"
date: "2019-02-22T13:25:00.284Z"
categories: [go]
comments: true
---

Throughout 2018 I was exploring several ideas that could be turned into small products. I'm often most productive while traveling, and ended up building an MVP for one of my simpler ideas in a single day. Let's look at the concept, some code, and how I went from idea to product so quickly.

<!--more-->

### What is an MVP?

An MVP is the **"minimum viable product"** that can make your idea work. The bare bones solution to a problem. Only the absolute necessary parts of your app that you need in order to validate market fit.

**In other words, you want the fastest way to find out if your idea has merit or not.**

So one way to do this is to build an MVP with the bare minimum of features that can let people use your app as soon as possible.

You also want them to be able to give feedback on it in some way. This is critical! It could be as simple as a form field to leave their email address. If they do that, you have feedback: Someone is interested in your idea.

And you have their contact info, so once your product evolves you can try to get them to sign up.

### From Idea to MVP

Last year I was quite active trading [Decentraland](https://decentraland.org/) parcels. 

For those who are not familiar, Decentraland is a virtual world built on the Ethereum blockchain. A large space made up of individual parcels that can be traded for real money. They are used by developers to build Virtual Reality experiences. These parcels tend to go for thousands of dollars and there is a whole community focused on trading them.

*Before you go spending money on this though: I believe Decentraland is far from finished and most of the trading happens among a small group of users. I merely use it as an example, due to my next point:*

The marketplace for these parcels was lacking in features. In particular, I wanted to be able to be alerted as soon as a parcel was listed under a certain price.

![Coding at the pool](/img/posts/mvp-at-the-pool.jpg)

So one day while I was traveling and sitting at a pool in Thailand, I decided to code up a quick tool to let me do just that. And before I even got started, I came up with a whole bunch of features that I wanted to add: 

- I wanted to be able to set price ranges.
- It should be possible to filter parcels by criteria like adjacency to districts or roads.
- Sometimes you want a parcel in a specific area or near certain coordinates.
- While we're at it, specific parcels should be monitored too.

So if I'm adding all these features, I might as well let users sign up for this service. Then I could add even more features and charge for them. As an example premium feature: Monitor the underlying *smart contract* instead of the marketplace API, to get faster results.

I did some research and it turned out that there was at least one other service similar to this. But it was private, invite-only, and not very far along either. People were occasionally talking about it in the trading community, so at least there was some demand. So somebody had already kind of validated the market for me.

I decided that I could build this for myself and see if anyone was interested in using it. But because it's such a small target market, I didn't want to spend a lot of time making something nobody else wanted. So I defined what I needed in order to have a minimum viable product:

- I should be able to enter an email address, so that the app can send alerts to it.
- It should be possible to set a price target - anything below will be of interest.
- As soon as the tool finds a new listing, send an alert.

That's all we need to have something that works, solves our initial problem and enables feedback through email signups.

### The Code

Honestly this is the least interesting part. The process of finding and defining your MVP is much more critical.

I used Go for this tool. Having a micro service running in the background to query the marketplace API is what Go is great at. And serving a small API to facilitate email signups is easy, too. Not to forget that deploying a binary to [Heroku](https://heroku.com) is very simple.

The app is split in two parts: A worker that will periodically query the marketplace, and a small web API to handle emails.

The worker's main loop looks like this:

``` go
deals := make(map[string]Deal)

	for {
		ticker := time.NewTicker(time.Duration(pollingIntervalInMinutes) * time.Minute)
		rateLimit := make(chan struct{})
		for _, site := range sites {
			select {
			case <-ticker.C:
				go parseAuctions(site, &deals)
			case <-rateLimit:
				ticker.Stop()
				return
			}
		}
	}
```

The above code makes it so that we run `parseAuctions` every couple of minutes and don't send too many requests at once.

So in the `parseAuctions` function we connect to the marketplace API and look for deals:

``` go
func parseAuctions(site Site, deals *map[string]Deal) {
	log.Info("Looking for latest LAND deals...")
	req, err := http.NewRequest("GET", site.URL, nil)
	if err != nil {
		log.Error(err)
	}
	req.Header.Set("Content-Type", "application/json")

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		log.Error(err)
	}

	defer resp.Body.Close()

	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Error(err)
	}

	var responseBody ResponseBody
	err = json.Unmarshal(body, &responseBody)
	if err != nil {
		log.Error("There was an error:", err)
	}

	newDeals := make(map[string]Deal)
	for _, parcel := range responseBody.Data.Parcels {
		if parcel.Publication.Price < dealPrice {
			if _, found := (*deals)[parcel.ID]; !found {
				log.Info("New deal found! Adding deal with ID: ", parcel.ID)
				name := "Parcel"
				if parcel.Data.Name != "" {
					name = parcel.Data.Name
				}
				deal := Deal{
					ID:        parcel.ID,
					X:         parcel.X,
					Y:         parcel.Y,
					Name:      name,
					Price:     parcel.Publication.Price,
					Timestamp: parcel.Publication.BlockTimeCreatedAt,
				}
				// TODO: refactor this
				newDeals[parcel.ID] = deal // only used for email alert
				(*deals)[parcel.ID] = deal // keep deals in memory
			}
		}
	}
	if len(newDeals) > 0 {
		sendAlert(&newDeals)
	}
	log.Info("Finished looking for deals. Sleeping...")
}
```

If any deals are found, `sendAlert` will send emails to every subscriber in our database, with a list of deals and a link to each one. I use [Sendgrid](https://www.sendgrid.com) for email delivery.

And the entire API to handle email signups fits in 150 lines of code, complete with double opt-in.

``` go
// omitted imports and var declarations for brevity

func subscribeHandler(w http.ResponseWriter, r *http.Request) {
	email := r.PostFormValue("email")
	log.Info("Email signed up: ", email)

	sendConfirmationEmail(email)

	w.Header().Set("Content-Type", "text/html; charset=utf-8")
	fmt.Fprint(w, "Thanks for signing up! We've sent you a quick email to confirm your subscription. Please check your spam folder in case you didn't get it, or contact us at contact@example.com.")
	return
}

func confirmHandler(w http.ResponseWriter, r *http.Request) {
	email := r.URL.Query()["email"][0]
	token := r.URL.Query()["token"][0]

	if token == hmac256(email, doubleOptInSecret) {
		w.Header().Set("Content-Type", "text/html; charset=utf-8")
		row := db.QueryRow("SELECT * FROM users WHERE email=$1", email)

		var user User
		err := row.Scan(&user.ID, &user.Email)
		if err != nil && err != sql.ErrNoRows {
			log.Error(err)
			return
		}

		if err == sql.ErrNoRows {
			if _, err := db.Exec(fmt.Sprintf("INSERT INTO users (email) VALUES ('%s')", email)); err != nil {
				log.Error("error: %q", err)
			}
			log.Info("Email confirmed: ", email)
			fmt.Fprint(w, "Success! We'll start sending you alerts whenever our script finds a nice deal.<br><br>You can unsubscribe anytime using the link in any of our emails. We're planning to add more features soon to let you customise alerts.")
			return
		}

		fmt.Fprint(w, "This email is already in our list.")
		return
	}

	w.Header().Set("Content-Type", "text/html; charset=utf-8")
	fmt.Fprint(w, "There was something wrong with your email confirmation link. Please let us know at contact@example.com if the problem persists.")
	return
}

func unsubscribeHandler(w http.ResponseWriter, r *http.Request) {
	// omitted for brevity
}

func sendConfirmationEmail(to string) {
	token := hmac256(to, doubleOptInSecret)

	// omitted for brevity

	message := mail.NewSingleEmail(from, subject, mail.NewEmail("", to), plainTextContent, htmlContent)
	client := sendgrid.NewSendClient(sendgridAPIKey)
	response, err := client.Send(message)
	if err != nil {
		log.Error(err)
	} 
}

func hmac256(str string, secret string) string {
	h := hmac.New(sha256.New, []byte(secret))
	h.Write([]byte(str))
	return hex.EncodeToString(h.Sum(nil))
}

func main() {
	// omitted env vars for brevity

	var err error
	db, err = sql.Open("postgres", os.Getenv("DATABASE_URL"))
	if err != nil {
		panic(err)
	}
	defer db.Close()

	if _, err := db.Exec("CREATE TABLE IF NOT EXISTS users (id serial primary key not null, email varchar not null unique)"); err != nil {
		log.Error("Error creating database table: %q", err)
	}

	http.HandleFunc("/subscribe", subscribeHandler)
	http.HandleFunc("/unsubscribe", unsubscribeHandler)
	http.HandleFunc("/confirm", confirmHandler)
	if err := http.ListenAndServe(fmt.Sprintf(":%s", port), nil); err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}
```

As you can see there is a lot of room for improvement. Most of the features I wanted are missing. Emails could be handled much better. Deals are kept in memory instead of a database, so if we restart the worker, old deals will be found again.

But those are details. The important part is that the service is working and sending emails correctly. I used this myself for several months, first locally and then deployed to Heroku.

### So What Happened to the Project?

Frankly I just never finished a landing page for it. I never showed the app to anyone. And I lost interest in Decentraland and trading parcels.

But I practiced how to take an idea and deploy the most basic version of it, from start to finish. I used third-party services like Sendgrid and Heroku to cut some corners. 

And I realized that a landing page is just as important as the MVP itself, if not more. 

Because as soon as you start getting the word out about your app, you will be held accountable for it. Which is why you should get that out of the way as soon as possible. To make it easier for yourself to stick with the project. And to get feedback before you even start building.
