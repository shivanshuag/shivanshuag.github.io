+++
Categories = ["Hacking"]
Description = ""
Tags = ["cowsay", "fortune", "gre"]
date = "2014-12-24T13:35:54+05:30"
menu = "main"
title = "Learning English with Cowsay"
+++

Linux has some absurd tools like fortune and cowsay. Fortune is a simple utility which displays random fortune cookies every time the user opens the terminal. Cowsay puts the message you give to it as an argument in a speech bubble with an ASCII image of a cow. You can combine cowsay and fortune to display a fortune cookie in the speech bubble of a cow. Nothing useful if you take them at face-value.

I have been repeatedly failing in my attempts to study English for GRE. I had tried Magoosh GRE flashcard, Barron's Gre Book but kept forgetting the words. I decided to display words instead of fortune cookies in my terminal. I use my computer profusely so this might help.

First problem was to get a list of words. Someone has uploaded all the words from Magoosh GRE flashcard and Barron's GRE on quizlet.com [here](http://quizlet.com/45492734/magoosh-gre-flashcards-basic-common-advanced-all-1000-flash-cards/) and [here](http://quizlet.com/8997815/barrons-800-essential-words-for-the-gre-flash-cards/). Now it was just a matter of scraping them and a simple python script was apt for the task. I scraped all the words and their meanings in a text file named `words`, each word separated by a '%'. This '%' has a special significance. It is useful in creating a random access file for strings which is used by the fortune program.

{{< highlight bash >}}
strfile words
{{< /highlight >}}

This generates a `words.dat` file. I moved both - the `words.dat` file and the `words` file to the folder that contains the fortune database which in my case is `/usr/share/fortune`. Now fortune also uses this file for selecting cookies. But I just want the words and not any other cookie. In `fortune`, you can specify the name of the database from which you want it to randomly select a string. So, `fortune words` did the task.

While exploring fortune, I also found that you can specify the probabilities with which it selects string from each database. This is helpful beacuse I want to to select words from Magoosh Basic and Magoosh Common list more frequently than Magoosh Advanced. For this, I need the words in separate files according to category. No probelm because I found the different lists on quizlet.com [here](http://quizlet.com/45473963/magoosh-gre-flashcards-basic-all-i-vii-flash-cards/), [here](http://quizlet.com/45473977/magoosh-gre-flashcards-common-all-i-vi-flash-cards/) and [here](http://quizlet.com/45473953/magoosh-gre-flashcards-advanced-all-i-vii-flash-cards/), scraped them into different files and followed the same procedure. The script used for scraping and the list of words is available in a github repository [here](https://github.com/shivanshuag/cowsay-gre). So now my command is

{{< highlight bash >}}
fortune fortune 40% magoosh_basic 40% magoosh_common magoosh_adv
{{< /highlight >}}

This gives a word from Basic and Common list with a probability of 0.4 and from Advanced list with a probability of 0.2.

To make it more interesting, I piped the output to cowsay. In cowsay, you can specify different ASCII images as argument. There is a default set of images in the `/usr/share/cows` folder.

{{< highlight bash >}}
fortune -c 40% magoosh_basic 40% magoosh_common magoosh_adv | cowthink -f $(find /usr/share/cows -type f | shuf -n 1)
{{< /highlight >}}

This command selects a random ASCII image from the set of images in `usr/share/cows` folder for the cowsay application. I added this to my `.zshrc` file. And the results were interesting!

     _________________________________________ 
    ( (/usr/share/fortune/magoosh_basic) %    )
    ( dispatch noun: the property of being    )
    ( prompt and efficient                    )
    (                                         )
    ( Synonyms : despatch , expedition ,      )
    ( expeditiousness                         )
    (                                         )
    ( She finished her thesis with dispatch,  )
    ( amazing her advisors who couldn't       )
    ( believe she hadn't written 60 scholarly )
    ( pages so quickly.                       )
    (                                         )
    ( verb: dispose of rapidly and without    )
    ( delay and efficiently                   )
    (                                         )
    ( As soon as the angry peasants stormed   )
    ( the castle, they caught the king and    )
    ( swiftly dispatched him.                 )
    (                                         )
    ( This word has other definitions but     )
    ( these are the most important ones to    )
    ( study                                   )
     ----------------------------------------- 
              o           \  / 
               o           \/  
                   (__)    /\         
                   (oo)   O  O        
                   _\/_   //         
             *    (    ) //       
              \  (\\    //       
               \(  \\    )                              
                (   \\   )   /\                          
      ___[\______/^^^^^^^\__/) o-)__                     
     |\__[=======______//________)__\                    
     \|_______________//____________|                    
         |||      || //||     |||
         |||      || @.||     |||                        
          ||      \/  .\/      ||                        
                     . .                                 
                    '.'.`                                

                COW-OPERATION                           



     _________________________________________ 
    ( (/usr/share/fortune/magoosh_basic) %    )
    ( pinnacle noun: the highest point        )
    (                                         )
    ( Synonyms : acme , elevation , height ,  )
    ( meridian , peak , summit , superlative  )
    ( , tiptop , top                          )
    (                                         )
    ( At its pinnacle, the Roman Empire       )
    ( extended across most of the landmass of )
    ( Eurasia, a feat not paralleled to the   )
    ( rise of the British Empire in the 18th  )
    ( and 19th century.                       )
     ----------------------------------------- 
             o
              o
                ^__^ 
        _______/(oo)
    /\/(       /(__)
       | W----|| |~|
       ||     || |~|  ~~
                 |~|  ~
                 |_| o
                 |#|/
                _+#+_



     _________________________________________ 
    ( (/usr/share/fortune/magoosh_basic) %    )
    ( aboveboard adjective: open and honest   )
    (                                         )
    ( Synonyms : straightforward              )
    (                                         )
    ( The mayor, despite his avuncular face   )
    ( plastered about the city, was hardly    )
    ( aboveboard - some concluded that it was )
    ( his ingratiating smile that allowed him )
    ( to engage in corrupt behavior and get   )
    ( away with it.                           )
     ----------------------------------------- 
            o    ,-^-.
             o   !oYo!
              o /./=\.\______
                   ##        )\/\
                    ||-----w||
                    ||      ||

                   Cowth Vader






    _______________________________________ 
    ( (/usr/share/fortune/magoosh_adv) %    )
    ( hauteur noun: overbearing pride       )
    ( evidenced by a superior manner toward )
    ( inferiors                             )
    (                                       )
    ( Synonyms : arrogance , haughtiness ,  )
    ( high-handedness , lordliness          )
    (                                       )
    ( As soon as she won the lottery, Alice )
    ( begin displaying a hauteur to her     )
    ( friends, calling them dirty-clothed   )
    ( peasants behind their backs.          )
     --------------------------------------- 
          o                _
           o              (_)   <-- TeleBEARS
            o   ^__^       / \
             o  (oo)\_____/_\ \
                (__)\  you  ) /
                    ||----w ((
                    ||     ||>> 



     _______________________________________ 
    ( (/usr/share/fortune/magoosh_basic) %  )
    ( imponderable adjective: impossible to )
    ( estimate or figure out                )
    (                                       )
    ( According to many lawmakers, the huge )
    ( variety of factors affecting society  )
    ( make devising an efficient healthcare )
    ( system an imponderable task.          )
     --------------------------------------- 
        o
         o
        ^__^         /
        (oo)\_______/  _________
        (__)\       )=(  ____|_ \_____
            ||----w |  \ \     \_____ |
            ||     ||   ||           ||
