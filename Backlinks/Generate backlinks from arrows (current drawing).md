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

	if (files.length > 0) {
		// Remove current ctime property from all notes in .ex folders
		await Promise.all(
			files.map(async (file) => {
				const abstractFile = app.vault.getAbstractFileByPath(file.path);
				await app.fileManager.processFrontMatter(abstractFile, (frontmatter) => {
					if (frontmatter[currentCtime]) {
						delete frontmatter[currentCtime];
					}
				});
			})
		);
	}

    // Initialize backlink counter
    let counter = 0;

    // Process view elements to generate backlinks
    const viewElements = ea.getViewElements();
    if (viewElements) {

		// get arrows that have bindings
        const arrowElements = viewElements.filter(el => el.type === "arrow" && el.startBinding && el.endBinding);
		// get embeddable elements
        const embeddableElements = viewElements.filter(el => el.type === "embeddable");

        if (arrowElements.length > 0 && embeddableElements.length > 0) {
            // Generate arrow data
            const arrowData = arrowElements.map((arrow) => {
				const startLink = embeddableElements.find(el => el.id === arrow.startBinding.elementId)?.link?.replace(/\[\[|\]\]/g, "").replace(/\.md/g, "");
				const endLink = embeddableElements.find(el => el.id === arrow.endBinding.elementId)?.link?.replace(/\[\[|\]\]/g, "").replace(/\.md/g, "");

				// Check if arrow head is in the reverse direction <-
				const isReverse = arrow.endArrowhead == null && arrow.startArrowhead;
				// if the arrowheads are both null or both existent, make the connection mutual
				const isMutual = (arrow.startArrowhead == null && arrow.endArrowhead == null) || (arrow.startArrowhead && arrow.endArrowhead);

				return {
					arrowId: arrow.id,
					filePath1: isReverse ? (endLink ? endLink + ".md" : null) : (startLink ? startLink + ".md" : null),
					backlinks1: isReverse ? (startLink || null) : (endLink || null),
					filePath2: isMutual ? (endLink ? endLink + ".md" : null) : null,
					backlinks2: isMutual ? (startLink || null) : null,
				};
            }).filter(item => item.filePath1 && item.backlinks1);

            // Combine arrow data into a dictionary
            const combinedData = arrowData.reduce((acc, item) => {
				// check if filePath1 already exists, if not create an entry
                if (!acc[item.filePath1]) {
                    acc[item.filePath1] = { filePath: item.filePath1, backlinks: [] };
                }
				// add backlinks1 data to filePath1
                acc[item.filePath1].backlinks.push(`[[${item.backlinks1}|${item.arrowId}]]`);

				// check if filePath2 and backlinks2 are present in the item to add
                if (item.filePath2 && item.backlinks2) {
					// check if filePath2 already exists, if not create an entry
                    if (!acc[item.filePath2]) {
                        acc[item.filePath2] = { filePath: item.filePath2, backlinks: [] };
                    }
					// add backlinks2 data to filePath2
                    acc[item.filePath2].backlinks.push(`[[${item.backlinks2}|${item.arrowId}]]`);
                }

                return acc;
            }, {});

            // Update frontmatter with combined data
            const combinedDataArray = Object.values(combinedData);
			// update property (named after ctime) with backlinks
            await Promise.all(
                combinedDataArray.map(async (data) => {
					const filePath = data.filePath;
                    const abstractFile = app.vault.getAbstractFileByPath(filePath);
					const numberOfBacklinks = data.backlinks.length;
                    await app.fileManager.processFrontMatter(abstractFile, (frontmatter) => {
						console.log(`${filePath}: adding ${currentCtime} (backlinks: ${numberOfBacklinks})`);
                        frontmatter[currentCtime] = data.backlinks;
                        counter += numberOfBacklinks;
                    });
                })
            );
        }
    }

    alert(`${counter} backlinks generated.`);
})();