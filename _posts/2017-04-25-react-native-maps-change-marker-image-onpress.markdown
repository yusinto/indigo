---
published: true
title: "Change marker image onPress in react-native-maps"
layout: post
date: 2017-04-25 09:30
tag:
- react
- native
- maps
- change
- marker
- image
- onpress
blog: true
---

Recently I started a new pet project which involves maps and markers on ios and android. Of course I started
this new project in react native. I was expecting a somewhat challenging times ahead because it has been
a few months since I last did react native development. The landscape has definitely improved, and I feel
so fortunate to be a javascript developer at this present moment because of awesome tools like react-native
at our disposal. The possibilities are truly endless.

Airbnb has open sourced [react-native-maps](https://github.com/airbnb/react-native-maps){:target="_blank"} which 
made it so easy to integrate mapping capabilities with your app. There are steps to follow to set it all up but 
it's not that hard.

## The problem
Need to display custom map markers on react-native-maps. Then, onPress of a 
marker, change that marker image so the user can see it has 
been selected. The problem is there is no direct way to get the ref of 
the selected marker. Even if there is, there is no setImage method to
change the marker image.

## The solution
Out of the box, there's already an onPress event handler with MapView.Marker
which is a good starting point. We will use this along with the ref and 
image props (also supported out of the box) to solve our problem.

## Are you done talking? Show me some code!
So first things first, you need to install and link react-native-maps:

{% highlight js %}
// stick with 0.13.0 to avoid unresolved issues in ^0.14.0
yarn add react-native-maps@0.13.0
{% endhighlight %}

{% highlight js %}
react-native link react-native-maps
{% endhighlight %}

GOTCHA: the latest react-native-maps require babel-plugin-module-resolver as well
otherwise you'll get this error: Unknown plugin module-resolver

So do this:
{% highlight js %}
yarn add babel-plugin-module-resolver
{% endhighlight %}

Then we can write some code to render a basic map like this:

{% highlight javascript %}
const headers = {
  Accept: '*/*',
  'Content-Type': 'application/json',
  Authorization: 'your-api-key',
  'accept-encoding': 'gzip, deflate'
};
const body = JSON.stringify([{
  op: 'replace',
  path: '/environments/test';,
  value: true,
}]);
const url = 'https://app.launchdarkly.com/api/v2/flags/default/your-key';
const response = await fetch(url, {
    method: 'PATCH',
    headers,
    body
});

{% endhighlight %}

Of course you'll need to add some defensive programming for error catching
and retries plus configuration for test and production environments
plus notifications when updates are successful/not successful, and the
list goes on.
 
If you go down this path, you soon realise that this is not a trivial 
task by any means. An ad-hoc solution like this involves hard coding
flag names and continual updates which are almost as bad as waking up at 
12:01 AM to do the deployments manually.

Enter [ld-scheduler](https://github.com/yusinto/ld-scheduler){:target="_blank"}.

## ld-scheduler
With ld-scheduler, you do this from your node app:

{% highlight js %}
yarn add ld-scheduler
{% endhighlight %}

then

{% highlight javascript %}
import ldScheduler from 'ld-scheduler';

ldScheduler.runEveryXSeconds({
  environment: 'test',
  apiKey: 'your-secret-api-key',
  slack: 'your-slack-webhook-url'
});
{% endhighlight %}

and you schedule your flags through launch darkly's dashboard:

![LaunchDarkly dashboard scheduling config](/assets/images/ld-scheduler-flag-settings-resized.png)

**HACK**: We hijack the description field to store our scheduling config as a json object where:
<ul>
    <li>taskType is killSwitch</li>
    <li>value is true (kill switch on) or false (kill switch off)</li>
    <li>
        targetDeploymentDateTime must be in YYYY-MM-DD HH:mm Z
        <p>
            <b>NOTE:</b> the UTC offset at the end is especially important because ld-scheduler uses moment which will use the host's timezone if it is not specified.
             That means if you deploy ld-scheduler to the cloud say on aws lambda where the machine clock is set to UTC timezone, then your flag will be deployed at
             UTC time, which is probably not what you want unless you are living in London!
        </p>
    </li>
    <li>description is a textual string for the purpose of human readability</li>
</ul>

***AND*** you need to set a tag called "${yourEnv}-scheduled". For example, if you are scheduling a flag in the test environment,
your tag should be called "test-scheduled". Likewise if you are scheduling it in production, you need to add a "production-scheduled" tag.

When ld-scheduler runs, it will set your flag on/off according the the json configuration. It will also remove the "${yourEnv}-scheduled" tag so
it does not get reprocessed. If there's no other scheduled tags, then ld-scheduler also sets the "Description" field
to the json.description string, thereby deleting the json config replacing it with the description string.

This way, you can safely run 2 instances of ld-scheduler; one for each environment without having to worry about race conditions.

## Extra
ld-scheduler supports a second taskType "fallThoughRollout" which you can use to set the default fallThrough rollout percentage:

{% highlight json %}
{
    "taskType": "fallThroughRollout",
    "targetDeploymentDateTime": "2017-03-3 02:33",
    "description": "Human readable flag description",
    "value": [
        {
            "variation": 0,
            "weight": 90000
        },
        {
            "variation": 1,
            "weight": 10000
        }
    ]
}
{% endhighlight %}
where variation 0 is true and variation 1 is false. Weight is in mili-percentage (if there's such a word) i.e. 90000 === 90% and 10000 === 10%.
Of course you would enter this json object in the "Description" field of your flag settings in launch darkly's dashboard
***AND*** set a "${yourEnv}-scheduled" tag.

## Conclusion
Check out the [sample code](https://github.com/yusinto/ld-scheduler/tree/master/example){:target="_blank"} for a working example and let me know if this is useful (or not)!

---------------------------------------------------------------------------------------
