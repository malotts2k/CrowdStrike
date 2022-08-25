# CrowdStrike EDR - an example of my prior work responsibilties while using CrowdStrike Falcon EDR

Crowdstrike Falcon is an enterprise-grade endpoint detection and response security product that helps security and incident response engineers identify potential threats on their networks. It is especially prolific at helping its engineers hunt threats and distinguish between detected threats and benign events, more commonly known as false positives. 

Using the CrowdStrike Hunting guide, I bolted together a query that would help with the team's threat hunting efforts. This query helped identify malicious processes, connectivity, and behavior.

Here is the query that I commonly used to key in on common threats and cut through a lot of the noise.

![image](https://user-images.githubusercontent.com/105020710/186570254-e26e5652-7a79-405f-9f35-0306cf801121.png)

The initial results of running this query regularly yielded thousands of events.

![image](https://user-images.githubusercontent.com/105020710/186570425-f5c3e2d5-aa97-4b76-b479-74e2bf3aefa7.png)

Altering the query to group common commands with the count helps cut down a lot of the non-relevant results.

![image](https://user-images.githubusercontent.com/105020710/186570930-d8f8872f-5ae6-48e5-a4df-00644e0cd2b4.png)

This also shows how frequently each command is being run. 

![image](https://user-images.githubusercontent.com/105020710/186571228-71d610dc-3dab-4a17-bd8b-d841329e5fe6.png)

The more common commands can typically be ignored since they usually signify benign tasks. We want to zero in on the unique tasks. The less frequent the process, the more suspect it is.

For instance, the process shown below warrants suspicion for a few reasons:

![image](https://user-images.githubusercontent.com/105020710/186571489-909374bf-cae9-4c7c-85ce-501e8405d29a.png)

It's an executable called at.exe, it seems to be attempting to schedule a task each weekday and it's passing through another command called escalate.exe. This process certainly warrants further attention:

![image](https://user-images.githubusercontent.com/105020710/186571674-660a5210-80c8-4ad8-87af-a099c87c79a5.png)

To dig further we need to dig into the event information where relevant data is revealed like the IP address of the remote system and the hash of the executable.

![image](https://user-images.githubusercontent.com/105020710/186571906-e59e0148-b2ac-4e8b-b9fa-5046714445d4.png)

The beauty of CrowdStrike lies with its elegant interfaces and pivot processes that lets you seemingly go another layer deep. Here we will select the option to use the Explorer view.

![image](https://user-images.githubusercontent.com/105020710/186572092-48d27c63-65b4-44fa-a78a-378c8f2f0df4.png)

The process tree shows where the command to schedule the task was called by A.EXE:

![image](https://user-images.githubusercontent.com/105020710/186572299-240cf451-a0d8-4e32-b497-a4afee0526d0.png)

So what exactly sticks out about this shady event that we haven't already gleaned from looking at the event details?

Let's take a further look at the command line data in the screenshot.

![image](https://user-images.githubusercontent.com/105020710/186572470-f2c7b8a1-a378-430e-a79b-a0316fdaf00e.png)

Things that immediately stick out:

* Single letter executable names are not common in most environments.
* Executables are not typically stored in the documents folder.
* The accepteula parameter is suspicious - it is common with sysinternals. This may be indicating that it's trying to use psexec.
* escalate.exe is not a known tool or something that the infrastructure team is known to leverage.

![image](https://user-images.githubusercontent.com/105020710/186573547-2aa7f5ca-78a8-4b8d-8ac8-146e83160761.png)


As shown in the above screenshot, a.exe is shown as common both locally and globally so maybe inspecting the hash value will give us more details.

![image](https://user-images.githubusercontent.com/105020710/186573707-0278ac3d-4309-44d4-9181-0a4804b87fa2.png)

It appears the hash matched the sysinternals psexec and confirmed that a.exe was simply a renamed variation of this program.

So what does that really tell us? This is a copy of psexec that is trying to get elevated command line access to schedule a task that likely escalates account privileges. At this point I would open a formal incident in order to additionally investigate and remediate this confirmed threat.

