/*

```js
*/
(async () => {
    // Get current drawing path
    const currentPath = ea.targetView.file.path;
    // Fetch current .ex file directly by path
    const currentExFile = app.vault.getAbstractFileByPath(currentPath);

    // Process view elements to generate backlinks
    const viewElements = ea.getViewElements();
    if (viewElements) {

		// get arrows that do not have bindings
        const arrowElements = viewElements.filter(el =>
            el.type === "arrow" && (!el.startBinding || !el.endBinding)
        );

        // if orphaned arrows exist, change their stroke color to red
        if (arrowElements.length > 0) {
            ea.copyViewElementsToEAforEditing(arrowElements);
            ea.getElements().forEach((el)=>{
                el.strokeColor = "#fa5252";
            });
            await ea.addElementsToView(false,true);
        }
        // alert the number of orphaned arrows found
        alert(`${arrowElements.length} orphaned arrows.`);
    }
})();