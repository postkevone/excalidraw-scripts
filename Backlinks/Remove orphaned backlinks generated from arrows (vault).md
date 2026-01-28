/*

```js
*/
(async () => {
    // Get all files in one call to minimize repeated calls to app.vault.getFiles()
    const allFiles = app.vault.getFiles();

    // Get all note files inside .ex folders
    const files = allFiles.filter(file => file.parent.name.includes(".ex") && file.extension === "md");

    // Return early if there are no 
    if (files.length === 0) {
        alert("No notes to process.");
        return;
    }

    // Get all .ex note files
    const exFiles = allFiles.filter(file => file.name.includes(".ex") && file.extension === "md");
    // Save .ex file ctimes as strings in a Set for fast lookup
    const exFilesCtimes = new Set(exFiles.map(file => String(file.stat.ctime)));

    // Counter for updated notes
    let counter = 0;
    
    // Process notes in .ex folders
    await Promise.all(
        files.map(async (file) => {
            const filePath = file.path;
            const abstractFile = app.vault.getAbstractFileByPath(filePath);

            await app.fileManager.processFrontMatter(abstractFile, (frontmatter) => {
                const properties = Object.keys(frontmatter);

                // Iterate and filter properties in a single step
                for (const property of properties) {
                    if (/^\d{13}$/.test(property) && !exFilesCtimes.has(property)) {
                        console.log(`${filePath}: removing ${property}`);
                        delete frontmatter[property];
                        counter++;
                    }
                }
            });
        })
    );

    // Alert the number of properties removed
    alert(`${counter} properties removed.`);
})();