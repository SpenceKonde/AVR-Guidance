# Libraries
### (and other codes you expect others to use, modify, play with, and learn from)
Writing libraries that you will make available to others and expect that they will actually use presents some particular concerns that code designed for use by only yourself or your collaborators dowes (even if you store it is a public github repo - a practice I wholeheartedly endorse). 

Work in progress

## Technical Matt3ers
When you write code for your own projects, even if you try to make it generally useful, you will naturally make assumptions about what it would support. But if you're making it for use by other people - particularly if it's popular and you listen to the ones who ask for hlep, you will rapidly discover that people are using your code in different contexts and on different hardware than you had in mind when you wrote it. To maximize it's usefulness (and minimize the nubmer of people asking questions), these are some guidelines

### Use #if / #ifdef liberally
If your code would work on many parts with only small changes, use #if and #ifdef to switch implementation details depending on the hardware they are using. Ideally, if you an determine that it wont work on that hardware, you can do something like `#error "This library does not support this chip, see the README for more information"`

### Test for peripherals, not chips
In the aforementioned #if's , test for pheripherals your code needs (by checking whether the register names are defined) not the specific parts you used or know off the top of your head will work. For example, if your library involves input capture, you might implement it on the 16-bit Timer1 peripheral on most classic AVRs have at least one of, and on a Type B timer on that the modern AVRs have, and then test for `#if defined(TCB0) || defined(TCCR1C)` - because if it had either of those things, you could be pretty sure your library would work fine there. 

## Non-technical Matters
There are also some issues that have nothing to do with the code itself that you need to consider. 

### This isn't your job (well, unless it is) 
If the code is not in support of a product you're selling, you are under no obligation to help people with it. Whem someone asks you to add some feature that you know will take a lot of work, and that you don't think is useful, don't hesitate to close it and say that this feature does not justify the time it would take, but that you'd welcome a pull request if they want to write it (assuming you actually would, at least). Sometimes you can get people to add features for you that way, too. 

### Your code reflects on you
TPeople will judge your personality and ability based on what they see in your code. Potential employers will see it when they search for you on social media, etc. So, don't be offensive or political in the comments (if nothing else, this ensures that you won't get preoplre contacting you just to complain about the political comment you made because you were listening to some political show or podcast in the backhround when you wrote it). The norms even within one country can vary more than many people realize - and the internet is global. That said, nobody is going to be particularly bothered by occasional profanity in a comment where it seems warranted, just be reasonable. 
Particularly if you are in the software field, or place particular value on other people thinking you are particularly competent at something, you also need to think about how the code reflects on your ability and programming habits. Inonsistent indentations, misspelled words in variable names amd things like that make you look sloppy, particularly in code you've made a point of documenting and making available for others. Doing things in a clumsy way - something that reveals that you don't know the Right Way to do something - that reflects on your knowledge of the language. If you are doing it in the more verbose and and "newbie" way for a reason, a comment to that effect is good (and also pre-empts the helpful user's pull request that replaces it with the more "graceful" way).
All these considerations apply to repos that are visible, but not optimized for use by others. If you have a mix of "released to public for re-use" repos and "something I did for myself, but might as well let people see it" it's a good idea to mention in the top of the README those in the latter category to the effect of "The code in this repo was made for personal projects, and is not expected to be useful to others (disabling issues would be a good idea in cases like that). 
