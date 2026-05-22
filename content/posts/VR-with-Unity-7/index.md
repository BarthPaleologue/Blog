---
title: "VR With Unity #7"
author: "Barthélemy Paléologue"
type: ""
date: 2024-01-27T13:16:11+01:00
subtitle: "Evaluation of the technique"
image: ""
tags: ["vr", "unity", "c#", "igd301"]
---

In the previous article, we developped a selection technique for a VR supermarket. Here is the result we achieved:

{{< youtube --iDxQKy9V0 >}}

You should probably read the previous article first: https://barthpaleologue.github.io/Blog/posts/vr-with-unity-6/

Today's blog post is all about evaluation. We will see how we can evaluate a selection technique, and how we can use the results to find ways to improve it.

## Evaluation

Before we start evaluating, we should ask ourselves what is it we want to evaluate, which metrics do we use?

For a selection technique, relevant metrics can be accuracy, speed, user sense of presence, user task workload and user enjoyment. We will use these three metrics to evaluate our technique. Finally we can get some general feedback from the users.

For this project, I needed to evaluate the technique on 3 people (including myself). The task was to select 20 items in the supermarket as fast as possible. Each participant did the task twice, one for training, and one for evaluation.

### Accuracy

How many errors then? Well, none actually. 

I wasn't expecting it to be perfect, but it was. Even the evil sandwiches from last time were selected without error! This is a good sign that the visual feedback system bear fruit.

### Speed

You could argue that they were so accurate because the technique is very slow. So how fast is it?

{{<figure src="total time.png" alt="Alt text" caption="Alt text" caption-position="bottom">}}

As you can see the average time for the entire task is around 100 seconds. As each participant had to select 20 items, we reach an average of 5 seconds per item. This is quite a good result when considering the users had to also move around the supermarket.

It would be interesting to test it against simple raycasting with teleportation, but I don't have more time to implement a second technique. Because of occlusion issues, I am very confident that the raycasting technique would be slower and less accurate.

### User Sense of Presence

"On a scale from 1 to 10 how present did you feel in the virtual world?” (1 lowest 10 highest)

The sense of presence is a qualitative metric that gives an idea of how much the user feels like he is in the virtual environment. The average score was 7.3 out of 10. It is hard to know if the technique had an impact on the score or if the environment played a bigger role.

### User task workload

“On a scale from 1 to 10, how easy was to perform the task” (1 lowest 10 highest)

The technique was found very easy to use with an average score of 9 out of 10.

### User enjoyment

“On a scale from 1 to 10, how much fun did you have during the task ” (1 lowest 10 highest)

The average score was 9.3 out of 10. The technique was found very fun to use. Maybe the background music I added helped a bit.

### User general feedback

The last metric is user feedback. This is a more qualitative metric that gives interesting insights to improve the technique.

The shelf grabbing was easy to understand and use, no participant had any issue with it. Item grabbing was also easy to understand, but a bit harder because the bounding boxes did not always match the shape of the item (see the sandwich example in the previous article). 

Teleportation was also found easy to use and understand.

The main issue was with shelf release. I did not mention explicitely that teleporting would release the shelf, and thus the feature was not discovered which lead to some frustration during training.

## Conclusion

Overall, the technique was found easy to use and very accurate. Users also had a lot of fun! The main issue was with shelf release, which could be fixed by adding a visual feedback when teleporting, or detecting when the user is trying to throw the shelf away.

This is the end of this series of articles. I hope you enjoyed it, and found it useful!

If you have any question, you can reach out to me by email at <a href="mailto:barth@paleologue.fr">barth@paleologue.fr</a>.
