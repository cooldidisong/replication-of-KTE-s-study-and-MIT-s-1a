# replication-of-KTE-s-study-and-MIT-s-1a
data reduction&amp; analyses

rm(list = ls())

# read.csv
s1<- read.csv("ToM.csv")

# delete some irrelated variable
s1 <-subset(s1,select = -c(DataFile.Basename,formalExpmovie.RESP,formalExpmovie.RT,keypress1answer))

# delete B- trials
library("dplyr")
s1 <- filter(s1,keypress2answer != "{ENTER}")

#delete wrong RESP
library("dplyr")
s1 <- filter(s1,KeyPress1RESP == "q")# delete wrong Keypress1RESP
s1 <- filter(s1,KeyPress2RESP == "p")# delete wrong Keypress2RESP


###……filter subjects whose responds out of 3000ms……###

#filter response2
library("dplyr")
s1 <- filter(s1,Keypress2RT > 33000 & Keypress2RT < 36000)

#filter response1

#dim comparison1 as P+A+
comparison1 = subset(s1,s1$Parray == "p" & s1$Aarray =="p")
comparison1 <- filter(comparison1,Keypress1RT>27900 & Keypress1RT<31000)
#dim comparison2 as P-A-
comparison2 = subset(s1,s1$Parray == "{ENTER}" & s1$Aarray =="{ENTER}")
comparison2 <- filter(comparison2,Keypress1RT>27900 & Keypress1RT<31000)
#dim comparison3 as P+A-
comparison3 = subset(s1,s1$Parray == "p" & s1$Aarray =="{ENTER}")
comparison3 <- filter(comparison3,Keypress1RT>21900 & Keypress1RT<25000)
#dim comparison3 as P-A+
comparison4 = subset(s1,s1$Parray == "{ENTER}" & s1$Aarray =="p")
comparison4 <- filter(comparison4,Keypress1RT>16900 & Keypress1RT<20000)

#combine all 4 comparison data
s <- rbind(comparison1,comparison2,comparison3,comparison4)
s1 <- s
# filter invalid sub
table(s1$Subject)
library("dplyr")
s1 <- filter(s1,Subject != c("10","12"))
s1 <- filter(s1,Subject != c("10","12"))
# calculate RT
reactionTime = s1$Keypress2RT - 33000
s1$reactionTime <- reactionTime
table(s1$formalExp)


--------------#### data preprocessing ####-------------------------

#### --- t-test for replications --- ####
library(lsr)
myCalculateCohensD <- function (comparison1, comparison2) {
  c1aggregate <- aggregate(reactionTime ~ Subject + Parray + Aarray, comparison1, mean)
  c1aggregate = c1aggregate[order(c1aggregate$Subject),]
  c2aggregate <- aggregate(reactionTime ~ Subject + Parray + Aarray, comparison2, mean)
  c2aggregate = c2aggregate[order(c2aggregate$Subject),]
  #c1aggregate$Subject == c2aggregate$Subject # to check that the ordering are the same.
  
  manualDiff <- c1aggregate$reactionTime - c2aggregate$reactionTime
  #stanDev <- sd(manualDiff)
  #tStat <- mean(manualDiff)/sd(manualDiff) * sqrt(length(manualDiff))
  cohenD <- mean(manualDiff)/sd(manualDiff); 
  return(cohenD)
}

myCalculateTTest <- function (comparison1, comparison2) {
  c1aggregate <- aggregate(reactionTime ~ Subject + Parray + Aarray, comparison1, mean)
  c1aggregate = c1aggregate[order(c1aggregate$Subject),]
  c2aggregate <- aggregate(reactionTime ~ Subject + Parray + Aarray, comparison2, mean)
  c2aggregate = c2aggregate[order(c2aggregate$Subject),]
  c1aggregate$Subject  == c2aggregate$Subject
  return(t.test(c1aggregate$reactionTime, c2aggregate$reactionTime, paired=FALSE))
}

#---- P-A- - P+A+ ----#
comparison1 = subset(s1, s1$Parray == "{ENTER}" & s1$Aarray == "{ENTER}")
comparison2 = subset(s1, s1$Parray == "p" & s1$Aarray =="p")
myCalculateCohensD(comparison1, comparison2)
myCalculateTTest(comparison1, comparison2)

#--------P+A- —— P-A- ---------#
comparison1 = subset(s1, s1$Parray == "p" & s1$Aarray == "{ENTER}")
comparison2 = subset(s1, s1$Parray == "{ENTER}" & s1$Aarray =="{ENTER}")
myCalculateCohensD(comparison1, comparison2)
myCalculateTTest(comparison1, comparison2)

#---- P-A+ - P-A- ----#
comparison1 = subset(s1, s1$Parray == "{ENTER}" & s1$Aarray == "p")
comparison2 = subset(s1, s1$Parray == "{ENTER}" & s1$Aarray =="{ENTER}")
myCalculateCohensD(comparison1, comparison2)
myCalculateTTest(comparison1, comparison2)

#---- P-A+ - P+A- ---- #
comparison1 = subset(s1, s1$Parray == "{ENTER}" & s1$Aarray =="p")
comparison2 = subset(s1, s1$Parray == "p" & s1$Aarray =="{ENTER}")
myCalculateCohensD(comparison1, comparison2)
myCalculateTTest(comparison1, comparison2)
