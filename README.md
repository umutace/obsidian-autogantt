If we can query tasks, why not automate a Gantt chart to show off the progress made on those tasks?

Forked from this template:
https://forum.obsidian.md/t/automatic-gantt-chart-from-obsidian-tasks-dataview/50512/10

Required Plugins:
- Tasks
- Templater
- Dataview

Features:
Almost the same "template" as above, but with some changes:
- Dataview Syntax (properties with `[property:: value]` instead of emojis.) version of the script
- tasks have to be tagged as #task
- template asks also the title of the chart and the format of the axis (like the scope of the chart)
    - only implemented in the Dataview version as of now.

what needs to be done:
- cleanup on both templates
- feature set matching on both templates

