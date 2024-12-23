---
showDone: false
---
```dataviewjs
function textParserDVFormat(task) {
    const properties = ["[start::", "[due::", "[scheduled::", "[created::", "[priority::", "[repeat::", "[completion::", "[cancelled::", "[onCompletion::"]

    let dueText = task.due;
    let scheduledText = task.scheduled;
    let startText = task.start;
    let addText = task.created;
    // Dataview syntax uses "completion" instead of "done" date!
    let doneText = task.completion;

    let priorityText = task.priority;

    // Get name without trailing properties
    const propertiesIndex = properties.map(property => task.text.indexOf(property)).filter(index => index >= 0);
    let words;
    if (propertiesIndex.length > 0) {
        words = task.text.slice(0, Math.min(...propertiesIndex)).split(" ");
    } else {
        words = task.text.split(" ");
    }
    // Remove the #task tag
    words = words.filter((word) => (word) !== "#task" && (word) !== "");
    // Put subsequent tags in []
    let newWords = words.map(
        (word) => word.startsWith("#") ? `[${word.slice(1)}]` : word);
    // Join the words back together
    let nameText = newWords.join(" ");

    // Dataview returns properties with Date automatically as epoch time!
    return {
        add: addText,
        done: doneText,
        due: dueText,
        name: nameText,
        priority: priorityText,
        scheduled: scheduledText,
        start: startText
    };
}

function loopGantt (pageArray, showDone){
    let queryGlobal = "";
    let today = new Date().valueOf();

	// Loop through the pages looking for tasks
    let i;
    for (i = 0; i < pageArray.length; i+=1) {
        let taskQuery = "";
        if (!pageArray[i].file.tasks || pageArray[i].file.tasks.length === 0) {
            continue;
        }
		// Add a section for the page if tasks are found
        queryGlobal += "section "+pageArray[i].file.name+"\n\n"
        let taskArray = pageArray[i].file.tasks;
        let taskObjs = [];
        let noteCreationDate = new Date(pageArray[i].file.ctime);
        var parentEndDate = {};
        // Loop through the tasks
        let j;
        for (j = 0; j < taskArray.length; j+=1){
            taskObjs[j] = textParserDVFormat(taskArray[j]);
            let theTask = taskObjs[j];
            if (theTask.name === "") continue; // Skip if task name is empty
            // Skip done tasks if showDone is false
            if (!showDone && theTask.done) continue;
            let startDate = theTask.start || theTask.scheduled || theTask.add || parentEndDate[taskArray[j].parent] || noteCreationDate || today;
            let endDate = theTask.done || theTask.due || theTask.scheduled;
            if (!endDate) {
                if (startDate >= today) {  // If start date is in the future
                    let weekLater = new Date(startDate);
                    weekLater.setDate(weekLater.getDate() + 7);
                    endDate = weekLater
                } else {  // If start date is in the past
                    endDate = today;
                }
            }
            parentEndDate[taskArray[j].line]=endDate;
            // Clean dates as strings
            let endDateString = new Date(endDate).toISOString().slice(0,10);
            let startDateString = new Date(startDate).toISOString().slice(0,10);
            if (theTask.done){
                taskQuery += theTask.name + `    :done, ` + startDate + `, ` + endDate + `\n\n`;
            } else if (theTask.due){
                if (theTask.due < today){
                    taskQuery += theTask.name  + `    :crit, ` + startDate + `, ` + endDate + `\n\n`;
                } else {
                    taskQuery += theTask.name  + `    :active, ` + startDate + `, ` + endDate + `\n\n`;
                }
            } else if (theTask.scheduled){
                if (startDate <= today){
                    taskQuery += theTask.name + `    :active, ` + startDate + `, ` + endDate + `\n\n`;
                } else {
                    taskQuery += theTask.name + `    :inactive, ` + startDate + `, ` + endDate + `\n\n`;
                }
            } else {
                taskQuery += theTask.name  + `    :active, ` + startDate + `, ` + endDate + `\n\n`;
            }
        }
        queryGlobal += taskQuery;
    }
    return queryGlobal;
}

const Mermaid = `gantt
        title <% await tp.system.prompt("Enter the name of this Gantt chart", "Tasks")%>
        dateFormat x
        axisFormat <% await tp.system.prompt("Enter the axis format", "%b %e")%>
        `;
let pages = dv.pages('<% await tp.system.prompt("Enter the scope (\"parent/folder\", #tag or [[page]])", true) %>');
// Assumes tasks with only #task tag are relevant for the Gantt chart.
// Use `let filteredPages = pages;` instead if tag is not used.
let filteredPages = pages.map(page => {
    let tasks = page.file.tasks.filter(t => t.text.includes("#task"));
    return {...page, file: {...page.file, tasks: tasks}};
});
let showDone = dv.current().showDone;
let ganttOutput = loopGantt(filteredPages, showDone);
//

//add dummy task so that today's date is always shown:
let today = new Date().valueOf();
ganttOutput += ". :active, " + today + ", " + today + "\n\n";

dv.paragraph("```mermaid\n" + Mermaid + ganttOutput + "\n```");

// Generate a link to view the resulting chart
dv.span("[View this chart in a browser](https://mermaid.ink/img/"+btoa(Mermaid+ganttOutput)+")")

// Print the ganttOutput to inspect it
//dv.paragraph(ganttOutput);
//dv.paragraph("```\n" + Mermaid + ganttOutput + "\n```");
```
