Issue Summary
Between April 3rd, 17:20 UTC and April 6th, 10:00 UTC, customers in the EU experienced elevated H10, H19, H21, and H26 errors. These errors accounted for less than 1% of the EU traffic during the impacted period. We sincerely apologize for any downtime arising from this issue and any adverse effects our customers experienced.

Here is some additional detail about what happened and steps we are taking to mitigate future outages of a similar nature.
Who was affected?

Approximately 12% of EU applications were affected at least once by this incident. These applications would have seen an increase in H10, H19, H21, and H26 errors on any web dynos, resulting in their app being unreachable. Overall, less than 1% of requests were affected during this incident.

Furthermore, only customers that deployed changes to or modified config vars on their applications in the EU were affected by this incident. Applications that did not generate any new releases were unaffected by this incident.
What Happened?

Timeline
During the scheduled EU maintenance on April 3rd, a remote procedure to reconfigure several instances timed out. This timeout went unnoticed leaving one instance configured incorrectly. This incorrect configuration meant that when customers performed actions resulting in web dynos being started and stopped, our routing fleet had a chance of missing the state updates leading to the errors mentioned above.

These missed state updates were very hard for us to discover because our routing fleet only maintains a connection to the affected class of instance for 30 minutes. After this time the connection is terminated and cycled to another server. This 30-minute window meant that during our investigation applications would exhibit the elevated H10 error codes, then begin working normally again, while other applications that had been previously normal would suddenly start raising H10 alerts. This caused us to follow false leads and believe that remediation actions were working when in fact they were not.

During the maintenance, we used a well-established automation tool to perform this maintenance so as to limit the possibility of human error. This tool has been used in similar maintenance operations previously without incident, so we trusted it.

To increase the security of our platform we recently implemented a strict timeout for human-driven system administration activities. We were unaware that this change had unintended effects on our existing automation tools, leading to the timeout condition described above. The problem was compounded further by the fact that our automation tool did not make this kind of timeout error visible to our engineers in any way.
What did we do to prevent wider impact?

As soon as we noticed elevated H10 errors, our engineers flushed routing caches. This remediation appeared to return the H10 errors to a standard level. However, the elevated rate of errors soon returned.

When the errors spiked again, our engineers continued their analysis of possible causes. Before proceeding with any further investigation, we took steps to flush caches again. We knew this action would not resolve the action, but it helped to reduce customer impact while we investigated other root causes. We repeated the act of flushing these caches throughout our investigations to minimize customer impact as much as possible.

Root cause and resolution
During our investigations, we suspected the database maintenance performed the night before might have been a cause. To rule this out, we took steps to revert the changes made during the earlier maintenance window. This rollback did not resolve the issue.

Eventually, our engineers were able to work backward from our internal logging and trace the issue to the single instance serving out-of-date routing data. With this confirmation, our engineers corrected the instance's configuration, resulting in resolution of impact.
What will we do to mitigate problems like this in the future?

Corrective and preventative measures
To mitigate future problems of this nature, we are taking several steps. First, we are examining the monitoring and alerting of these parts of our infrastructure and taking the necessary measures to ensure that it meets the high standards we hold in place for our systems.

Secondly, we will rectify the ambiguity around the H10 error. An H10 error should indicate that the web dyno has crashed. However, during this incident, our routing layer, confused by incorrect routing data, made an invalid assumption that apps were crashing when they were not. We will add logic to identify this case of incorrect routing data and return an H99 (“Platform Error”) instead.

We will also harden our automated tooling to work better with the our new secure timeouts. We will improve the visibility and monitoring of those timeouts to make it clearer when they might affect our maintenance plans and tooling.

Finally, we have launched a much deeper and thorough investigation into our handling of the response. We are exploring ways that we can coordinate our investigations to provide a much more timely remediation to incidents such as this one.
