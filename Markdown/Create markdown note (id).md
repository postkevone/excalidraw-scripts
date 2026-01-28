/*

```js
*/
(async () => {
    // Generate filename
    let fileName = Date.now().toString();

    // Get the current folder path
    const currentPath = ea.targetView.file.path.split("/").slice(0, -1).join("/");
    // Create target folder file
    const targetFolder = app.vault.getFolderByPath(currentPath);

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
                useObsidianDefaults: false,
                backgroundMatchCanvas: false,
                backgroundMatchElement: true,
                backgroundOpacity: 60,
                borderMatchElement: true,
                borderOpacity: 0,
                filenameVisible: false,
            }
        );
        await ea.addElementsToView(true, true, true);
    } catch (error) {
        alert("Failed to create a new markdown file.");
        console.error(error);
    }
})();