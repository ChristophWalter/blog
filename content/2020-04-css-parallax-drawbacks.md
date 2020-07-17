# Can you make this background have a such a cool parallax effect?

This questions (and the following requirement) lead to a small research and some experiments about the current state of parallax implementations. This article is about a simple parallax effect where the background scroll faster (or slower as the content).

It was pretty clear that using [CSS 3D transforms is the way to go](https://developers.google.com/web/updates/2016/12/performant-parallaxing) if you want a clean and performant solution. I really liked the approach until I tried it myself...

The project that was chosen to get a new parallax landingpage has a quite large codebase. So there is no greenfield. And this is where it gets tricky (as always):

 - Using parallax in a single page requires some adaptions at a global scale. We need a wrapper with fixed height on top level to avoid multiple scrollbars.
 - Even then the scrollbar is within this wrapper and so it looks different than on all the other pages without parallax.
 - In addition, a fixed header must be outside of the parallax construct to keep it sticky. So the page will not scroll while hovering over it.

These are no show stopper and you can come up with solutions that will work for you. But you need to think about the impact.

It is also valid to think about different technologies. There are several JavaScript libraries for parallax effects. In my experience it was way easier with less side effects to integrate one of them in a complex project. But you need to measure and decide if you can accept the drawback in performance.
