---
layout: post
title:  "Rewriting PHP scripts in Go"
date:   2017-02-05 20:18:00
categories: [go, php]
comments: true
---

Recently I was dealing with a migration job that was written in PHP to migrate a million user accounts and their associated data to a new DB and table structure. Originally this script was needed to migrate only a couple hundred accounts for a quick demo, so we didn't really care about its performance, but when we later tried to scale it up we ran into issues with memory as it often happens with long-running PHP scripts. Another problem was that it took way too long to run it.

The first step I took to improve this script was to get rid of Doctrine ORM calls. Using the ORM was fine for quickly getting the script done in the first place, but now it had to be replaced with very large prepared statements. Using the DBAL instead of relying on the ORM already improved performance drastically, but the script would still take around an hour to migrate all users and consume several GB of memory, still leaking quite a bit with each batch. And the same was true after going down another layer and using PDO directly.

I eventually ended up rewriting the entire script in Go, which not only took me less time than the time I had already spent trying to tweak the existing PHP script, but also immediately got rid of any performance problems.

Thanks to goroutines we can make multiple DB calls at the same time, dramatically speeding up the running time of the script. This can be tweaked with a command line flag to limit the number of concurrent goroutines to balance between memory usage (more goroutines, more memory) or a faster running script. Running only a few goroutines with our example takes only a couple dozen MB, while cranking it up to as much as our DB (in dev environment) can take will result in several hundred MB used and speed up the script to finish migrating 1 million accounts within 3 minutes.

In the following snippet I use a channel to limit the number of active goroutines to avoid opening too many DB connections. I use the waitGroup to make sure the script actually waits for all of its work to finish.

``` go
...
// limit number of goroutines to the value we specified on the command line
limit := make(chan struct{}, maxGoroutines)

// we get 'total' from a DB count query and 'batchsize' from a config flag
loops = int(math.Ceil(float64(total) / float64(batchsize)))

waitGroup.Add(loops)
for i := 0; i < loops; i++ {
	limit <- struct{}{} // will block if maxGoroutines is exceeded
	...

	go processBatch(offset, limit)
}

waitGroup.Wait() // otherwise main() will just finish without waiting for goroutines
...
```

All we do in the `processBatch` function is to query the data we need, map it to our Go structs and construct our prepared statements to insert into the new DB. Then at the end of the function we let our channel and waitGroup know that we are done. This could probably even be combined to only use one of the two.

``` go
func processBatch(offset int, limit chan struct{}) {
	...
	<-limit
	defer waitGroup.Done()
	...
}
```

The final script still leaves a lot of room for improvement, but I'm already quite happy with the result and plan to use Go for more of our migration scripts and background jobs in the future, as it seems a lot more suited for it than PHP.
