## An analytical model of overbooking

Let's start with a really simple model of optimal ticketing.  Let's establish the conceptual facts first:

* If the airline only sold as many seats as physically exist on the plane, there is some risk that flights would not be full because there is some positive probability that the number of people showing up for a flight will be less than the number of people who paid to be on that flight.
* The airline incurs a cost for each empty seat
* If the airline does sell more seats than it has and all ticketed passengers show up, the airline pays a penalty for each person who showed up over the number of seats on the plane.

My super simple example includes the following parameters:

$$t$$ = the price of the airline ticket
$$c$$ = the cost incurred by the airline for each empty seat
$$p(n)$$ = probability that exactly n people show up for the flight
$$s$$ = total seats on the plane
$$n$$ = number of tickets to sell.  
$$penalty$$ = bribe paid per bumped passenger if flight is overbooked.

**Note**

If a passenger holding a ticket for a flight gets bumped it is common practice to offer that passenger some cash plus a seat on the next flight.  Therefore, the cost to the airline of a bumped passenger is cash money they have to give out plus the opportunity cost of the seat on the next flight (i.e. they lose the ability to sell one seat on the next flight).  My model rolls these two things up into the same term.  


The airline's profit from selling $n$ seats is composed of three main parts:

1. The unconditional profit $nt$
2. The total cost of empty seats, $(s-k)*c$ if the number of people who show up for the flight, $k$ is less than the number of seats on the plane.
3. The total cost of bribing passengers, $(k-n)*penalty$, if the flight is overbooked.

The profit from selling $n$ tickets is:

$$\pi(n) = nt, if k=s$$
$$\pi(n) = nt - (k-n)*penalty, if k>s$$
$$\pi(n) = nt - (s-k)*c, if k<s$$

We can model the probability of all $n$ ticketed passengers showing up for a flight as a binomial distribution.  This approach essentially treats every individual as his/her own independent experiment and says that if the probability of a success ('showing up for the flight') on any one trial is $p$, then the probability that we will get $k$ successes in $n$ trials is:

$$p(x=k) = \frac{n!}{k!(n-k)!}p^{k}(1-p)^{n-k}$$

We can use this to solve the optimal overbooking problem by observing that expected profit for the airline is the profit under each passenger scenario (k people show up for a flight where n tickets were sold) weighted by the probability of each scenario.

A little more concrete: let's assume we have 30 seats on an airplane.  The expected value of selling 31 seats is:

$$p(1 out 31 people show up)*\pi(1) + p(2 out of 31 people show up)*\pi(2) + ... + p(30 out of 31 people show up)*\pi(30) + p(31 people show up)*\pi(31)$$

In this case $\pi(x)$ is the profit realized by the airline if $x$ people show-up for the flight.  

Now, the last piece we need for this example is the probability of success on any one trial. In our computational exercise we will experiment with the probability, but for now let's fix it at 0.98.  Then the expected value of selling 31 seats for a flight that has 30 seats is:

$$\sum_{k=1}^{k=31}=\frac{31!}{k!(31-k)!}0.98^{k}0.02^{31-k}\pi(k)$$


## When don't you overbook?

The first thing I want to look at is the following:

under what probability scenario is it never optimal (using our ad-hoc parameters) to overbook?

Quick refresher on the parameters: I'm going to set:

* total number of seats at 30
* cost per ticket at 300
* cost to the airline of each empty seat at 150 (half the ticket price)
* the probability that any one individual with a ticket for the flight shows up for the flight is 0.98

In this quick example I'm going to vary the per passenger overbooking penalty from 2X the ticket price to 15X the ticket price.  

```R
# a simulation model:

# function to compute the probability of k successes in n trials if the probability
# a success on any single trial is p
binom.fn <- function(n,k,p){
return((factorial(n)/(factorial(k)*factorial(n-k)))*(p^k)*((1-p)^(n-k)))
}

#function to calculate the profit conditional on k people showing up for a flight with
# s seats and a per passenger overbooking penalty of 'penalty', per passenger empty seat # cost of 'cost', and ticket price of 'ticket' 

pi.k <- function(s,k,penalty,cost,ticket){
  overbook <- abs(s-k)
  underbook <- max(0,s-k)
  
  pi <- (k*ticket) - (penalty*overbook) - (cost*underbook)
  return(pi)
}

# generalized function to calculate Expected profit of selling
Epi <- function(n,p,seats,tprice,bribe,cempty){
  k <- c(1:n)
  pk <- unlist(lapply(k,binom.fn,n=n,p=p))
  pik <- unlist(lapply(k,pi.k,s=seats,ticket=tprice,penalty=bribe,cost=cempty))
  E=sum(pk*pik)
  return(data.frame(n=n,p=p,seats=seats,ticket_price=tprice,bribe=bribe,
                    empty_seat_cost=cempty,EV=E))
}

#with an individual probability of 0.98 how high does the bumped passenger bribe
# need to be for it to be optimal NOT to overbook?
z <- rbind(
data.frame(rbindlist(lapply(c(30,31,32),Epi,p=0.98,seats=30,tprice=300,bribe=600,cempty=150))),
data.frame(rbindlist(lapply(c(30,31,32),Epi,p=0.98,seats=30,tprice=300,bribe=1800,cempty=150))),
data.frame(rbindlist(lapply(c(30,31,32),Epi,p=0.98,seats=30,tprice=300,bribe=3000,cempty=150))),
data.frame(rbindlist(lapply(c(30,31,32),Epi,p=0.98,seats=30,tprice=300,bribe=4500,cempty=150))))

ggplot(z,aes(x=n,y=EV,color=factor(bribe))) + geom_line() + geom_point() + 
  theme_bw()

```

INSERT PLOT HERE

As you can see, with these made up parameters, it is only optimal to sell exactly 30 tickets for a plane with 30 seats if the overbooking penalty is between 10X and 15X the ticket price.  

Why is overbooking such an attractive strategy with these parameters?  Well, one of the first things to note is that with 30 seats on a plane and a probability of 0.98 that each individual will show up for the flight, the flight is only actually full about 50% of the time.  

To see why this is, think about flipping a coin.  Instead of a fair coin that has a 50/50 shot of coming up heads, assume the probability of a coin flip producing 'Heads' is 0.98.  Now ask: *if we flipped this coin 30 times what is the probability that we would get 30 heads in a row?*

$$p(flip 1 = H & flip 2 = H...& flip 30 = H)=p(flip 1 = H)*p(flip 2 = H)*...*p(flip 30 = H)$$

Based on our probability of any flip coming up heads of 0.98 the probability of 30 heads in a row is:

$$0.98*0.98*0.98...*0.98=0.98^30=0.545$$

### What is the empty seat cost is REALLY low?

In the last example we found that with a probability that each individual will show up for his/her flight of 0.98 and an empty seat cost of $\frac{1/2}{ticket price}$ the only way it is optimal to not overbook is if the overbooking penalty is astronomically high (like between 10 and 15 times the ticket price).

Now I want to do a different sensitivity analysis and ask, *if the empty seat cost is really low, is it ever optimal to not overbook based on a passenger 'show-up' probability of 0.98?*

I won't leave you in suspense.  The answer is no.  Basically, there is no empty seat cost (at least not one that makes any sense) that makes it optimal to sell exactly 30 seats for a plane that has 30 physical seats available if the arrivale probability is 0.98 and the overbooking penalty is 10X the ticket price.

```R
# what if the empty seat cost is really low?
z <- rbind(
  data.frame(rbindlist(lapply(c(30,31,32),Epi,p=0.98,seats=30,tprice=300,bribe=3600,cempty=300))),
  data.frame(rbindlist(lapply(c(30,31,32),Epi,p=0.98,seats=30,tprice=300,bribe=3600,cempty=200))),
  data.frame(rbindlist(lapply(c(30,31,32),Epi,p=0.98,seats=30,tprice=300,bribe=3600,cempty=150))),
  data.frame(rbindlist(lapply(c(30,31,32),Epi,p=0.98,seats=30,tprice=300,bribe=3600,cempty=100))))

ggplot(z,aes(x=n,y=EV,color=factor(empty_seat_cost))) + geom_line() + geom_point() + 
  theme_bw()

```


INSERT PLOT HERE


## What if we vary the probabilities?

I thought it might be interesting to look at what probabilities we would need in order for exact booking (selling only the number of tickets as you have seats on the plane) to be optimal.  

For this I used a decent sized list of probabilities that varied by only small amounts.  The reason for this is that very small changes in the individual 'arrival probability' can produce pretty large changes in 'full flight' probability.  Just to illustrate this point a little more, here is a short list of how the arrival probability maps to odds of having a full flight:

$$p=0.98, p^{30} = 0.5454$$
$$p=0.985, p^{30} = 0.635$$
$$p=0.99, p^{30} = 0.739$$
$$p=0.995, p^{30} = 0.86$$

```R
# for different combinations of bribes and penalties, what probability 
# makes exact booking optimal?

b <- seq(from=600,to=2400,by=100)
c <- seq(from=30,to=300,by=50)
p.mesh <- c(0.98,0.9825,0.985,0.987,0.99,0.995,0.9975,1)

tmp.fn <- function(i,j){
return(
rbind(
data.frame(rbindlist(lapply(p.mesh,Epi,seats=30,n=30,tprice=300,bribe=b[i],cempty=c[j]))),
data.frame(rbindlist(lapply(p.mesh,Epi,seats=30,n=31,tprice=300,bribe=b[i],cempty=c[j]))),
data.frame(rbindlist(lapply(p.mesh,Epi,seats=30,n=32,tprice=300,bribe=b[i],cempty=c[j]))))
)  
}

df.list <- list()
for(j in 1:length(c)){
  tmp <- data.frame(rbindlist(lapply(c(1:length(b)),tmp.fn,j=j)))
df.list[[j]]<-tmp
}

z <- data.frame(rbindlist(df.list))


prob.filter <- z %>% group_by(bribe,ticket_price,seats,empty_seat_cost,p) %>%
                  arrange(p,n,seats,ticket_price,bribe,empty_seat_cost,-EV) %>%
                  mutate(diff1=EV-lead(EV)) %>%
                  filter(n==30 & diff1>0) %>%
                  group_by(n,seats,ticket_price,bribe,empty_seat_cost) %>%
                  summarise(p=min(p))

ggplot(prob.filter, aes(x=empty_seat_cost, bribe)) + geom_tile(aes(fill = p),
          colour = "white") + scale_fill_gradient(low = "white",
          high = "steelblue") +
  theme_bw() + scale_x_continuous(breaks=c) + xlab('empty seat cost')

```


INSERT TILE PLOT HERE

Here are a couple of quick observations on the tile plot:

* At low levels of empty seat cost (around one-tenth of the ticket price) and low level of overbooking penalty (around 2 times ticket price) an arrival probability of 0.995 is required in order for $n = 30$ to be optimal.
* Keeping empty seat cost low (still one-tenth of ticket price), $n = 30$ is optimal with arrival probability of 0.985 but only if the overbooking penalty is around 4 times the ticket price
* The arrival probability making $n=30$ optimal with overbooking penalty at 2 times ticket price is never less than 0.995 (86% change of having a full flight if number of tickets sold equals number of seats on the plane).


# A few final comments

This analysis could be made considerably more rigorous with a few tweaks:

*  airlines don't usually sell all the tickets for a particular flight at the same price.  So it would probably make sense to build in some price discrimination ($x$ number of first class seats at $px$, $y$ number of coach seats at $py$, etc.)
* airlines also don't know exactly how many seats they can sell on any flight.  They do (probably) have some idea of the elasticity of demand for tickets on a particular flight with respect to price.  That is, they have some idea of how responsive demand.  To make this example really cool we could build in some uncertainty on the part of the airline regarding how many tickets they can for a flight at a certain price.  
* An enterprising young economist could also probably set up a cool simulation experiment by making the overbooking penalty uncertain.  Specifically, I think it might be cool to let the overbooking penalty have some fat tails so that it was low on average but had some non-zero probabilities of extreme penalties.  It might be interesting to see how that impacts the airlines expected profit maximization problem.

Even as an overly simplistic toy problem I think what I have provided here has some value.  I was just thinking this morning that I can't even remember being on a flight where there wasn't some call for volunteers to give up a seat at the gate.  Meaning, my priors were that overbooking was a really, really common strategy.  I think my example here shows why: the probability of each individuals showing up for a flight they have a ticket for has to be really close to 1 (like 0.995, 0.997) in order for overbooking to be suboptimal.  
