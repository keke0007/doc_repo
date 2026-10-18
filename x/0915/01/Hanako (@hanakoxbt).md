---
author: "Hanako"
handle: "@hanakoxbt"
source: "https://x.com/hanakoxbt/status/2099269941509726404"
date: 2026-08-23T13:19:56.000Z
type: tweet
---
# Hanako (@hanakoxbt)

Workflows vs. Graphs, clearly explained!  
  
workflows are great, and almost everyone has one. here is the ceiling:  
  
a workflow decides every step before it runs.  
  
you drew eight boxes in March. six months later the same three fire, every single time, and the other five have never once been reached.  
  
then a case arrives that nobody drew, and it goes to the closest wrong box. quietly, with a green status, because from the inside that looks exactly like success.  
  
Graph engineering fixes this by moving the decision: not what the steps do, but when the steps get chosen.  
  
you need both, and here is the sentence that resolves the whole confusion:  
  
a workflow decides the steps before it runs. a graph decides them while it runs.  
  
↳ drawn in advance: the boxes, the branches, the order, the error path  
↳ decided at runtime: how many units exist, what each one is allowed to see, which ones get created at all  
  
Prompts → Context → Harness → Loops → Graphs  
  
branches do not make it a graph.  
  
the branches were drawn in advance too, which means every one of them is a case you already thought of.  
  
the trick is knowing which part is allowed to be fixed.  
  
the node kinds are fixed. a splitter is a splitter, a gate is a gate, a merge is code. what is not fixed is how many of them exist this run, and that is decided after something has been read.  
  
one thing to know before you scale it.  
  
a workflow fails in a way that never pages anyone.  
  
↳ the wrong branch ran, every check inside it passed, and the output is well formed  
↳ nothing errored, because routing to the wrong box is not an error, it is a route  
  
that last one catches careful people. you cannot test your way out of it either, because the test suite was written from the same diagram that has the gap in it.  
  
and the one that eats whole nights: a workflow that has never surprised you is not stable, it is narrow.  
  
if it has run four hundred times and produced the same three shapes, it is not handling your work. it is handling the part of your work that fits it, and you have quietly stopped sending it the rest.  
  
below i have quoted my full guide on graph engineering. it covers the three topologies, the verifier patterns, and where the gate should actually open.  
  
save this and read it below ↓

![🎥 Video](<Hanako%20(@hanakoxbt).assets/image-01.jpg>)

> **Hanako (@hanakoxbt)**
> 
> ![Article cover](<Hanako%20(@hanakoxbt).assets/image-02.jpg>)
> 
> 📝 **Loops and Graphs: how to stop babysitting agents and only approve the last step (full course)**
> 
> You check every step your agents take. Not because you want to, but because nothing else does.
> Getting out of that chair takes two things, and almost everyone builds only one of them.
> A loop makes a...