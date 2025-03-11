# Complete Discord Quest
Complete Discord Quest Without Streaming Or Playing the Game 

> [!NOTE]
> This no longer works in the browser! If you need to use a browser instead of the desktop app, use an extension to add the string `Electron/` anywhere in your user agent.
> 
How to use this script:
1. Accept the quest under User Settings -> Gift Inventory
2. Join a vc
3. Stream any window (can be notepad or something)
4. Press <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>I</kbd> to open DevTools
5. Go to the `Console` tab
6. Paste the following code and hit enter:
```js
let wpRequire;
window.webpackChunkdiscord_app.push([[ Math.random() ], {}, (req) => { wpRequire = req; }]);

let api = Object.values(wpRequire.c).find(x => x?.exports?.getAPIBaseURL).exports.HTTP;
let ApplicationStreamingStore = Object.values(wpRequire.c).find(x => x?.exports?.default?.getStreamerActiveStreamMetadata).exports.default;
let QuestsStore = Object.values(wpRequire.c).find(x => x?.exports?.default?.getQuest).exports.default;
let encodeStreamKey = Object.values(wpRequire.c).find(x => x?.exports?.encodeStreamKey).exports.encodeStreamKey;
let sleep = ms => new Promise(resolve => setTimeout(resolve, ms));

let quest = [...QuestsStore.quests.values()].find(x => x.userStatus?.enrolledAt && !x.userStatus?.completedAt)
if(!quest) {
	console.log("You don't have any uncompleted quests!")
} else {
	let streamId = encodeStreamKey(ApplicationStreamingStore.getCurrentUserActiveStream())
	let secondsNeeded = quest.config.streamDurationRequirementMinutes * 60
	let heartbeat = async function() {
		console.log("Completing quest", quest.config.messages.gameTitle, "-", quest.config.messages.questName)
		while(true) {
			let res = await api.post({url: `/quests/${quest.id}/heartbeat`, body: {stream_key: streamId}})
			let progress = res.body.stream_progress_seconds
			
			console.log(`Quest progress: ${progress}/${secondsNeeded}`)
			
			if(progress >= secondsNeeded) break;
			await sleep(30 * 1000)
		}
		
		console.log("Quest completed!")
	}
	heartbeat()
}
```
7. Keep the stream running for 15 minutes
8. You can now claim the reward in User Settings -> Gift Inventory!

You can track the progress by looking at the `Quest progress:` prints in the Console tab or by reopening the Gift Inventory tab in settings. The progress should update every 30s.
Make sure you invite ur alt account or ur friend in the VC and no need to do anything
## FAQ

**Q: Ctrl + Shift + I doesn't work**

A: To Use it , Open Run Command and type "%appdata%/discord" open settings.json with Notepad and add this "  "DANGEROUS_ENABLE_DEVTOOLS_ONLY_ENABLE_IF_YOU_KNOW_WHAT_YOURE_DOING": true, " between it in between 
and save it and go to properties of that json file and mark it as read only and close and reopen the discord , and it should work


**Q: I get an error saying "Unauthorized"**

A: Discord has patched the script from working in browsers. Use the desktop app, or find some extension which lets you change your User-Agent and append the string `Electron/` anywhere in it


**Q: I get a different error**

A: Make sure you've started streaming *before* running the script

