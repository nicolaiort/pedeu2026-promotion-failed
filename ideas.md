# Ideas

These are the basics on how this talk got to be.

## What even is a platform

Well that's a hard question. A question that people way smarter than me already elaborated in various talks, papers and youtube videos.
But nontheless we have to talk about it because otherwise what would we even be staging?

The TAG APP DELIVERY released their whitepaper "platforms" in march of 2023 and provides a definition as definitive as it is flexible.

> A platform for cloud-native computing is an integrated collection of capabilities defined and presented according to the needs of the platform’s users. It is a cross-cutting layer that ensures a consistent experience for acquiring and integrating typical capabilities and services for a broad set of applications and use cases. A good platform provides consistent user experiences for using and managing its capabilities and services, such as Web portals, project templates, and self-service APIs.

Source: https://tag-app-delivery.cncf.io/whitepapers/platforms/

The TL;DR: A collection of capabilitites providing a consistent way to access and combine services for building and operating applications.
So something kinda application centric?

It depends! The people who decide the name of departments at larger companies seemingly love to call everything a platform: "Out app platform" (Cloud foundry), "The kubernetes platform" (Kubernetes as a Service), "Our AI Platform" (Jupiter Notes)

So is my kubernetes platform that combines kubevirt with hosted control planes and workers as vms really a platform?
I'd say yes but others might disagree. But this question already includes one important part: A list of components have their own versions and probably need staging for updates.

At the end of the day a platfgorm not just a single - clearliy defined thing: It's composed of multiple things.
Things we have to stage.

## And what is a stage

Yes another philosopical question.
In some cases this could be just another namespace in a kubernetes cluster, a whole cluster on the same infra or even a system on seperated infra.
Simmilarly to the platform question, this will depend on your point of view and scale of your platform.

In most scenarios a stage is another deployment of the same platform/stack that runs seperated from the "production platform" but tries to emulate the prod as closely as possible.
The more operations-focused the platform admins and architects then to get, the deeper the seperation.
In my personal experiences most try to reach ultimate seperation by using dedicated hardware and networks.

## How many stages do I need

Grab as many as you can carry kid!
But most of the times there tend to be at least three basic stages: DEV, QA and PROD
Why? Because those are the stages our developers want to successfully stage and test their applications.

But now a second party joins the call: You, the platform engineer. You probably want at least one stage that you can use to pre-test changes without developers crying that they can't continue development because you took down their precious dev cluster with the database used by all developers.
So we add another stage: Let's call it LAB - you might allow or even encourage users to deploy stuff to that stage to get early integraion feedback. To be honest: I highly advise to getting some kind of real-world workload on your platform-testing stage. Otherwise you might as well test updates in a ephemeral environment because you won't be able to notice problems that might occur when updating an existing platform instead of deploying a fresh one.
If you're really advanced you could make this a 1:1 copy of another stage (DEV or QS) and even mirror traffic to your lab stage to run blackbox response tests.

And now your infra team heard about that thing you're running on their precious servers.
They would really like to get another stage that they can use to test firmware updates against running workload.
You might be able to combine this with your own testing stage, but there might be reasons why that could cause problems.

Of course this is a game that can be played until hardware or your favourite aws runs dry.
But you shouldn't. I'd suggest that anything over $Count_{DEV Stage}+2$ is too many stages, otherwise you're basicly running stages to run stages and not to actually seperate and test things.

## Technical Staging

Finally the part you've all been waiting for: The actual staging
Well kinda: The first thing we have to talk about is state management.
Otherwise we wouldn't know how to promote changes.

### All things gitops

Congratulations, you managed to configure every version and setting through some kind of file in a git repo.
For the sake of our argument let's just say that you're using argo or flux to sync a bunch of yaml files containing kubernetes resources or helm values.
Now the first question is: how do you want to structure your repo?

1. One folder per Stage with duplicated Files and changes get promoted manually or through a dedicated tool like the ArgoCD GitOps Promoter that can do automatic promotions based on health indicatiors
2. One global folder that contains the main manifests that just get build with something like kustomize to apply stage specific values (like domains)
3. A hybrid of both that manages common stuff in one folder but still has one additional folder per stage for specific stuff. (maybe you only deploy some debugging tools to lab because they would be a security risk when combined with real data)


Honestly they are all valida approaches: If you want to discuss the merits of these approaches (and even more variants) I hightly recommend the flux docs on how to structure your repository (https://fluxcd.io/flux/guides/repository-structure/)

But wait a minute: Option two sound just like a custom state file per stage

### Statefile

At the end of the day, most platforms i had the pleasure of working with ended up with one thing: A statefile
Sometimes it was calles, state.yaml, sometime versions.json and i event once saw a platform.xml but at the end of the day they all achived the same thing: One file that defined the version numbers of all parts of the platform.
The cool kids even automaticly promoted changes, but most start with a simple promotion mechanism: Edit the damn file, push to main and pray.

### The actual git release 

At this time we don't just bndle everything together in one state file, bt we actally use git releases or rather, since releases are more of a GitHub thing, tags in Git.
This means that we combine versions with configurations enabling a strong cohesion and correlation between version and config changes making sure there are no "but first we need to update this config because stuff got deprecated" situations.
And since everything is  just one large git tag, we can even treat our release process for the platform in the same way as any other software. We just need some sort of pipeline - or gitops - that applies it to one stage after another. Of course this eiter requires every stage to be a 1:1 copy of the last one or extracting some configurations - like env vars - out of the release bundle.
Or you could build a -lab, -dev, -qs and finaly a "latest" release one after another with everyone containing the information for all releases. But now the tags are only fancy decorators (but isnt that what they really are mean to be in git)?

## Promotion strategies

And the second part of staging: The actual process of moving changes from one stage to another.
As always there are a bunch of different methods and opinions.
These are some of the most common ones.

### Just DIY

Promote to a stage, test manually and copy over to the next stage if no one complains.
That is an approach as old as time and 80% of the time it works all of the time.
But if you ever tell this to your favourite compliance person they will come for you, so better run fast.

The manual approach is just too error prone to be a good idea for a long time and does not enforce any mandatory waiting periods. If you forget about it or copy over the wrong config on a friday you can kiss your weekend goodbye.

### Automagic™ Tests

If this was your first idea: You're probably from an software engineering background.
Most would call this the ideal staging mechanism: A suite of automated tests that check every possible angle of your platform, all APIs and edge cases. Simple divinity.

Of course it is a feverdream and also a waste of engineering time to build tests for every aspect of your platform. (Just ask your programmer friend on their oppinion about 100% Codecoverage for unit tests)
But building a solid end to end tests that checks all steps involved in deploying a typical application to your platform is a big confidence bump. And as a nice sideeffect you can also use it as a canary for monitoring your platform.
There are a thousand ways of actually acomplasing this from raw bash to dedicated frameworks.

But you want this, trust me.

### Static Delays

The answer of "enterprise architects" that havn't touched anything that is not a policy word document in the last 20 years.
(No front to architects that actually know what they're doing but i belive we all had the pleasure of encountering someone that never had any technical know-how or simply lost touch).
It's also the simplest next step after migrating away from the manual approach: A cronjob that just copies over changes from one stage to the next after a set number of days.
You might be fammiliar with orders like.

```mermaid
graph LR
    LAB-->|One day|-->DEV-->|2 days|QS-->|5 days|PROD
```

And don't get me wrong: They are a better baseline than nothing and have supported platforms I interaced with for years.
They just tend to become too ridgid for emergiencies (think a 10/10 CVE in your favorite ingress controller that enables remote code exec with cluster-admin rights) to we tend to fall back to a manual override.
And no one gurantees that every interaction with the platform will happen in the X days till promotion. Maybe no one ordered a new certificate because the current ones are still valid so you just learned about a breaking issuer change in production, "fun!".

### Ok so what should i do?

Combine the multiple approaches.
Start with implementing the delay based strategy with an option of fast-track important fixes/updates.
Then extend them with a daily e2e-test that simply blocks promotion if failed or a pre-promotion, your choice of order and method. Keep adding imporant interfaces to these automated checks until you can basicly be sure that even onbly one day of delay and testing between stages inspires confidence.

### But what if stuff breaks anyways?

Rollback, duh!
Ok that might be a bit more complicated than it sounds but make sure you can allways either rollback changes or quickly restore a fresh backup. How? That depends on your platform.


## Would someone think of the ~~children~~ Users

Often overlooked and the source of many ticket: Keeping your users up to date.
And I don't mean a weekly change/patch time where everyone has to expect some kind of minor disruption and pray that they configured their pod disruption budgets correctly.
Don't get me wrong: Knowing when doom will come is nice, but knowing what awaits us is even better.

One way of handling this is some kind of internal blog or slack channel with update/change announcements and - and this is the imporant part - an impact rating.
If we're talking Kubernetes as a service again most users won't care if you update your CCM to a new patch version that fixes a cosmetic bug.
But they will care about a breaking change that includes some nessecary changes on their part - even or especially if it's a simple as updating loadbalancer annotations (don't ask me how i ended up with this example).

An important addition is some sort of live overview wich version is deployed where.
"Just look at the git repo" some might say: "Have you ever heard of user experience" I'd answer and start some kind of war.
But honestly: Providing them a simple dashboard that just looks at your GitOps Resources/Statefile or Git Tags can make a difference like night and day when it comes to the amount of open issues regarding versions.
If you really want to make things transparent: Include an indicator of when changes are expected to arrive in the next stage.
Event if it's "just" a "+x Days" note in the dashboard this hugely increases transparency and will reduce the numer of time that impatient users will ask you when the new feature they have been waiting for will finally hit prod.

And in thse modern time you don't need to be some kind of frontend wizard to build a dashboard.
You can just ask your favourite LLM to help you build something out of your existing staging logic and a bit of tailwindcss.
And I can vouch for that approach, I actually did this just a couple of months ago in a larger enterprise environment building the basics of an extendable dashboard that just collects state from different repos using bash any yq in a moringing session.
To be fair: I knew exacly what technologies I wanted to use because i actually have a bunch of web dev experience. But nowadays almost everyone I know that works with dev-facing platforms knows some kind of web/frontend stack.
You might not be completely up to date but that's what the LLM is for. Just don't overcomplicate it and keep everything clean so you - or someone else - is still able to extend/change things afterwards.
But this is the point where i have to stop this sidequest before I get kicked off stage in the midst of a multi-hour rant about devops people actually knowing dev and AI assisted coding.

## TL;DR

Please stage your platform including all dependencies (if possible).
Please check what is actually part of your platform (it might be more than you think).
Use some kind of version control.
Try to use automatic tests and automatic promotions but keep a manual override for emergencies.
Remember to tell your users about changes/updates.