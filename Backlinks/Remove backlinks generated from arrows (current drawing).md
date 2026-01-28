/*

```js
*/
(async () => {
    // Get current drawing path
    const currentPath = ea.targetView.file.path;
    // Fetch current .ex file directly by path
    const currentExFile = app.vault.getAbstractFileByPath(currentPath);
    // Get ctime and convert to string for comparison
    const currentCtime = String(currentExFile.stat.ctime);

    // Get all notes inside .ex folders
    const allFiles = app.vault.getFiles();
    const files = allFiles.filter(file => file.parent.name.includes(".ex") && file.extension === "md");

    if (files.length === 0) {
        alert("No notes to process.");
        return;
    }

    // counter for removed backlinks
    let counter = 0;

    // Process notes concurrently
    await Promise.all(
        files.map(async (file) => {
            const filePath = file.path;
            const abstractFile = app.vault.getAbstractFileByPath(filePath);

            await app.fileManager.processFrontMatter(abstractFile, (frontmatter) => {
                const properties = Object.keys(frontmatter);

                // Remove properties that match currentCtime
                properties.forEach((property) => {
                    if (property === currentCtime) {
                        console.log(`${filePath}: removing ${property}`);
                        delete frontmatter[property];
                        counter++;
                    }
                });
            });
        })
    );

    // Alert the number of properties removed
    alert(`${counter} properties removed.`);
})();