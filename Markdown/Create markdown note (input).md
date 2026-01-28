/*

```js
*/
(async () => {
    // Prompt for note name
    let fileName = await utils.inputPrompt("New note", "Title");
    if (fileName) {
        // Trim unnecessary whitespace
        fileName = fileName.trim().replace(/\s+/g, " ");

        // Check if string is not empty
        if (fileName.length > 0) {
            // Get the current folder path
            const currentPath = ea.targetView.file.path.split("/").slice(0, -1).join("/");
            // Create target folder file
            const targetFolder = app.vault.getFolderByPath(currentPath);

            // check if folder has md notes inside
            if (Array.isArray(targetFolder.children)) {
                // Check for duplicate filenames inside folder
                const hasDuplicate = targetFolder.children.some(
                    child => child.extension === "md" && child.basename.toLowerCase() === fileName.toLowerCase()
                );

                if (hasDuplicate) {
                    alert(`"${fileName}" already exists.\nPlease choose another title.`);
                    return;
                }
            }

            try {
                // Create new markdown file
                const markdownFile = await app.fileManager.createNewMarkdownFile(targetFolder, fileName);

                // style settings
                ea.style.strokeColor = "#7e7e7e";
                ea.style.fillStyle = "solid";
                ea.style.strokeWidth = 4;
                ea.style.strokeStyle = "solid";
                ea.style.roughness = 0;
                // Add as embed note
                ea.addEmbeddable(0, 0, 500, 500, `[[${currentPath}/${fileName}]]`, markdownFile,
                    {
                        useObsdsidianDefaults: false,
                        backgroundMatchCanvas: false,
                        backgroundMatchElement: true,
                        backgroundOpacity: 60,
                        borderMatchElement: true,
                        borderOpacity: 0,
                        filenameVisible: true,
                    }
                );
                await ea.addElementsToView(true, true, true);
            } catch (error) {
                alert("Failed to create a new markdown file.");
                console.error(error);
            }
        }
    }
})();