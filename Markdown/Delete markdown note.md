/*
```js
*/
(async () => {
    // Get selected items into object named el
    const el = ea.getViewSelectedElements();
    if (el?.length) {
        // Get the current path prefix
        const currentPath = ea.targetView.file.path.split("/").slice(0, -1).join("/")+"/";

        // Loop through elements and process deletions
        const filesToDelete = el
            .filter(item => item.link && item.link.startsWith(`[[${currentPath}`))
            .map(item => item.link.replace(/\[\[|\]\]/g, "").replace(/\.md$/, ""));

        // Delete files if they exist
        for (const filePath of filesToDelete) {
            const markdownFile = app.vault.getAbstractFileByPath(`${filePath}.md`);
            if (markdownFile) {
                await app.vault.trash(markdownFile, false);
            }
        }

        // Delete the elements from the drawing
        ea.deleteViewElements(el);
    }
})();