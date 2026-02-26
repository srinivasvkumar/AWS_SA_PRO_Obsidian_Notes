```dataviewjs
// 1. Grab everything from all folders
let allFiles = dv.pages("")
    .where(p => !p.file.path.includes(".git")); // Still ignoring git junk

// 2. Map EVERYTHING by keyword (not just orphans)
let groups = {};
allFiles.forEach(f => {
    // Improved logic: skip common words like "AWS" to find the real topic
    let nameParts = f.file.name.split(/[ _-]/);
    let topic = (nameParts[0].toLowerCase() === "aws" && nameParts.length > 1) 
                ? nameParts[1] 
                : nameParts[0];

    if (!groups[topic]) groups[topic] = [];
    groups[topic].push(f.file.link);
});

// 3. Output as a clickable Index
for (let topic in groups) {
    if (groups[topic].length > 1) { // Only show topics that have multiple files
        dv.header(3, "Domain: " + topic);
        dv.list(groups[topic]);
    }
}
```