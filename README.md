# Laravel — Bester Practices

This is a commentary on https://github.com/alexeymezenin/laravel-best-practices 

When you're learning to code, the first step is to make code work. To accomplish something at all.
But once you create a slightly larger app, the code becomes either into a huge blob called the Ball of Mud.
Or, if you've already picked up some techniques to redirect the flow (e.g. functions or classes) and the DRY concept, you can even tangle it into some Spaghetti code!

The second step is to learn how to organize your code so that your project isn't total mess.
For that you usually learn various techniques like DRY, SOLID and so on.
At this point you should also learn how and when to apply these techniques.
Sadly, most of the time instead of teaching the tools along with their uses, you just get a huge toolbox with a "BEST PRACTICES" label.
The result? Devs at this point in their journey make their code either into a nearly infinite stack of layers (Lasagna code) or split it into a thousand isolated microclasses — Ravioli code.
The Ravioli is the ultimate application of the Single Responsibility principle: each dumpling is only responsible for one thing — invoking the next dumpling.

The reality is that the third step is to learn how and when to apply the techniques you've learned as a junior.
And I'll try to provide brief tips on that here.
Nothing comprehensive, just by giving a small commentary on each of the [best practices](https://github.com/alexeymezenin/laravel-best-practices) making them even bester.

> [!NOTE]  
> I only knew Spaghetti and Ravioli, but I asked ChatGPT for the rest of the so it must be correct.
> Even if it wasn't before this.
> It also suggested me other great names like Goulash code (a mix of all techniques) and Casserole code (rewritten so much it has become unrecognizable).
