# Troubleshooting Your Sketch
As developers, we inevitable spend most of our precious time chasing down bugs in our code. Often, this is simply an exercise in tedium - fixing every typo and spelling error as the compiler flags them up - or deep pondering when you realize your approach won't quite work after all, and you have to redesign some part of your program - but other times, figuring out what kind of bug you have, or even where it is, or why it's not working can be a challenge. This document describes some techniques that can be helpful for finding and/or understanding bugs.

There are two categories of problem - compile-time errors (compile errors). In this case, the 'verify' button will generate an error. If verify works, but 'upload' does not, there is an upload problem, not a compile problem, and the code may be fine. See (TODO: Write up ../UploadErrors)


If it doesn't compile, and you don't understand why, see (./CompileErrors)

If it compiles and uploads fine, but the sketch doesn't do what you want, that is a runtime problem; these can be the hardest problems to solve. (TODO: Start writing RuntimeIssue)

## It may not be where you think!
For either type of error, NEVER ASSUME YOU KNOW WHERE THE PROBLEM IS - you may think you do, if it's a compile error, the compiler tells you where it first noticed it. New Arduino users (and new programmers) - and sometimes experienced ones - often get stuck, usually with runtime problems, when they become convinced that the specific block of code related to the broken behavior is the cause of the problem. If you're sure the problem is in one part of the code, but you can't figure out why: Test your assumptions: Is it reaching the code you think has the problem? What paths is execution going down (add prints amd/or toggle pins connected to LEDs)? What values do variables have at key points in time? 

## No error - but "the old sketch is still there"
If it compiles, and says it uploaded, but the behavior looks to be unchanged from your previous version - **it isn't a weird upload problem, and the old sketch isnt "still there"** the IDE verifies that what's on the chip is the new code. (It is not even physically possible - there's nowhere other than the flash that a whole sketch worth of data could be stashed within the chip) It is a bug in your sketch, and the change you made actually did not alter the behavior you thought that it would. Either your sketch is exactly the same, or it acts exactly the same (if the size of the sketch is identical with and without that change, that is suspicious; if you "export compiled binary" with the two versions, and the files are the same, then the part of the code you're modifying is unreachable (due to a bug elsewhere), and the compiler realizes that and optimizes it out (see:UnexpectedOptimization). Some common causes of this are covered in (.RuntimeIssue).
