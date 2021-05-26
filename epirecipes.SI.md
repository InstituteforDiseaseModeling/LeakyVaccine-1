SI model of a vaccine trial
================
Josh Herbeck, revising a model by Simon Frost
2021-02-10

We consider a closed population of N initially susceptible people who
are assumed to be mixing randomly with contact rate c.

## simecol ODE model

dS/dt = -lambda*S dI/dt = lambda*S

Sp = susceptible placebo Ip = infected placebo

Sv = susceptible vaccinated Iv = infected vaccinated

Svh = susceptible vaccinated high exposure SvL = susceptible vaccinated
low exposure Ivh = infected vaccinated high exposure Ivl = infected
vaccinated low exposure

p = transmission rate (per contact) c = exposure rate (serodiscordant
sexual contacts per time) \#beta = p*c Prev = prevalence \#lambda = beta
* Prev epsilon = per contact vaccine efficacy; vaccine-induced reduction
in the risk of HIV infection from a single exposure

``` r
p = 0.01   #transmission rate (per contact)
c = 0.3   #contact rate (contacts per time)
beta <- p*c
Prev <- 0.20

lambda <- beta*Prev
epsilon <- 0.50 #per act vaccine efficacy
risk <- 3.0 #risk multiplier

parms <- c(lambda, epsilon, risk)
init <- c(Sv=1000,Iv=1,Sp=1000,Ip=1,Svh=500,Ivh=1,Svl=500,Ivl=1)
times <- seq(from=1, to=365*3, by=1)

sir_ode <- function(times, init, parms){
  with(as.list(c(init, parms)), {
  # ODEs
    
  # vaccine; homogeneous risk
  #Nv <- Sv+Iv
  dSv <- -lambda*epsilon*Sv
  dIv <- lambda*epsilon*Sv 
  
  # vaccine; heterogeneous risk
  #Nv.risk <- Svh+Ivh+Svl+Ivl
  dSvh <- -risk*lambda*(1-epsilon)*Svh
  dIvh <- risk*lambda*(1-epsilon)*Svh 
  dSvl <- -lambda*(1-epsilon)*Svl
  dIvl <- lambda*(1-epsilon)*Svl
    
  #placebo
  #Np = Sp+Ip
  dSp <- -lambda*Sp
  dIp <- lambda*Sp
  
  return(list(c(dSv,dIv,dSp,dIp,dSvh,dIvh,dSvl,dIvl)))
  })
}

sir_out <- lsoda(init, times, sir_ode, parms)
```

``` r
sir_out_long <- melt(as.data.frame(sir_out), "time")
sir_out_long <- sir_out_long[sir_out_long$variable %in% c('Iv', 'Ip','Ivl','Ivh'),]
```

``` r
ggplot(sir_out_long, aes(x=time/365, y=value, color=variable, group=variable))+
  #Add line
  geom_line(lwd=2) +
  #Add labels
  xlab("Years") + ylab("Infections")
```

![](epirecipes.SI_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->