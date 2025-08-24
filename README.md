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
<details>
	<summary>Click TO Expand</summary>

 ```js
delete window.$;
let wpRequire = webpackChunkdiscord_app.push([[Symbol()], {}, r => r]);
webpackChunkdiscord_app.pop();

let ApplicationStreamingStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.getStreamerActiveStreamMetadata).exports.Z;
let RunningGameStore = Object.values(wpRequire.c).find(x => x?.exports?.ZP?.getRunningGames).exports.ZP;
let QuestsStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.getQuest).exports.Z;
let ChannelStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.getAllThreadsForParent).exports.Z;
let GuildChannelStore = Object.values(wpRequire.c).find(x => x?.exports?.ZP?.getSFWDefaultChannel).exports.ZP;
let FluxDispatcher = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.flushWaitQueue).exports.Z;
let api = Object.values(wpRequire.c).find(x => x?.exports?.tn?.get).exports.tn;

let quest = [...QuestsStore.quests.values()].find(x => x.id !== "1248385850622869556" && x.userStatus?.enrolledAt && !x.userStatus?.completedAt && new Date(x.config.expiresAt).getTime() > Date.now())
let isApp = typeof DiscordNative !== "undefined"
if(!quest) {
	console.log("You don't have any uncompleted quests!")
} else {
	const pid = Math.floor(Math.random() * 30000) + 1000
	
	const applicationId = quest.config.application.id
	const applicationName = quest.config.application.name
	const questName = quest.config.messages.questName
	const taskConfig = quest.config.taskConfig ?? quest.config.taskConfigV2
	const taskName = ["WATCH_VIDEO", "PLAY_ON_DESKTOP", "STREAM_ON_DESKTOP", "PLAY_ACTIVITY", "WATCH_VIDEO_ON_MOBILE"].find(x => taskConfig.tasks[x] != null)
	const secondsNeeded = taskConfig.tasks[taskName].target
	let secondsDone = quest.userStatus?.progress?.[taskName]?.value ?? 0

	if(taskName === "WATCH_VIDEO" || taskName === "WATCH_VIDEO_ON_MOBILE") {
		const maxFuture = 10, speed = 7, interval = 1
		const enrolledAt = new Date(quest.userStatus.enrolledAt).getTime()
		let completed = false
		let fn = async () => {			
			while(true) {
				const maxAllowed = Math.floor((Date.now() - enrolledAt)/1000) + maxFuture
				const diff = maxAllowed - secondsDone
				const timestamp = secondsDone + speed
				if(diff >= speed) {
					const res = await api.post({url: `/quests/${quest.id}/video-progress`, body: {timestamp: Math.min(secondsNeeded, timestamp + Math.random())}})
					completed = res.body.completed_at != null
					secondsDone = Math.min(secondsNeeded, timestamp)
				}
				
				if(timestamp >= secondsNeeded) {
					break
				}
				await new Promise(resolve => setTimeout(resolve, interval * 1000))
			}
			if(!completed) {
				await api.post({url: `/quests/${quest.id}/video-progress`, body: {timestamp: secondsNeeded}})
			}
			console.log("Quest completed!")
		}
		fn()
		console.log(`Spoofing video for ${questName}.`)
	} else if(taskName === "PLAY_ON_DESKTOP") {
		if(!isApp) {
			console.log("This no longer works in browser for non-video quests. Use the discord desktop app to complete the", questName, "quest!")
		} else {
			api.get({url: `/applications/public?application_ids=${applicationId}`}).then(res => {
				const appData = res.body[0]
				const exeName = appData.executables.find(x => x.os === "win32").name.replace(">","")
				
				const fakeGame = {
					cmdLine: `C:\\Program Files\\${appData.name}\\${exeName}`,
					exeName,
					exePath: `c:/program files/${appData.name.toLowerCase()}/${exeName}`,
					hidden: false,
					isLauncher: false,
					id: applicationId,
					name: appData.name,
					pid: pid,
					pidPath: [pid],
					processName: appData.name,
					start: Date.now(),
				}
				const realGames = RunningGameStore.getRunningGames()
				const fakeGames = [fakeGame]
				const realGetRunningGames = RunningGameStore.getRunningGames
				const realGetGameForPID = RunningGameStore.getGameForPID
				RunningGameStore.getRunningGames = () => fakeGames
				RunningGameStore.getGameForPID = (pid) => fakeGames.find(x => x.pid === pid)
				FluxDispatcher.dispatch({type: "RUNNING_GAMES_CHANGE", removed: realGames, added: [fakeGame], games: fakeGames})
				
				let fn = data => {
					let progress = quest.config.configVersion === 1 ? data.userStatus.streamProgressSeconds : Math.floor(data.userStatus.progress.PLAY_ON_DESKTOP.value)
					console.log(`Quest progress: ${progress}/${secondsNeeded}`)
					
					if(progress >= secondsNeeded) {
						console.log("Quest completed!")
						
						RunningGameStore.getRunningGames = realGetRunningGames
						RunningGameStore.getGameForPID = realGetGameForPID
						FluxDispatcher.dispatch({type: "RUNNING_GAMES_CHANGE", removed: [fakeGame], added: [], games: []})
						FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
					}
				}
				FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
				
				console.log(`Spoofed your game to ${applicationName}. Wait for ${Math.ceil((secondsNeeded - secondsDone) / 60)} more minutes.`)
			})
		}
	} else if(taskName === "STREAM_ON_DESKTOP") {
		if(!isApp) {
			console.log("This no longer works in browser for non-video quests. Use the discord desktop app to complete the", questName, "quest!")
		} else {
			let realFunc = ApplicationStreamingStore.getStreamerActiveStreamMetadata
			ApplicationStreamingStore.getStreamerActiveStreamMetadata = () => ({
				id: applicationId,
				pid,
				sourceName: null
			})
			
			let fn = data => {
				let progress = quest.config.configVersion === 1 ? data.userStatus.streamProgressSeconds : Math.floor(data.userStatus.progress.STREAM_ON_DESKTOP.value)
				console.log(`Quest progress: ${progress}/${secondsNeeded}`)
				
				if(progress >= secondsNeeded) {
					console.log("Quest completed!")
					
					ApplicationStreamingStore.getStreamerActiveStreamMetadata = realFunc
					FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
				}
			}
			FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
			
			console.log(`Spoofed your stream to ${applicationName}. Stream any window in vc for ${Math.ceil((secondsNeeded - secondsDone) / 60)} more minutes.`)
			console.log("Remember that you need at least 1 other person to be in the vc!")
		}
	} else if(taskName === "PLAY_ACTIVITY") {
		const channelId = ChannelStore.getSortedPrivateChannels()[0]?.id ?? Object.values(GuildChannelStore.getAllGuilds()).find(x => x != null && x.VOCAL.length > 0).VOCAL[0].channel.id
		const streamKey = `call:${channelId}:1`
		
		let fn = async () => {
			console.log("Completing quest", questName, "-", quest.config.messages.questName)
			
			while(true) {
				const res = await api.post({url: `/quests/${quest.id}/heartbeat`, body: {stream_key: streamKey, terminal: false}})
				const progress = res.body.progress.PLAY_ACTIVITY.value
				console.log(`Quest progress: ${progress}/${secondsNeeded}`)
				
				await new Promise(resolve => setTimeout(resolve, 20 * 1000))
				
				if(progress >= secondsNeeded) {
					await api.post({url: `/quests/${quest.id}/heartbeat`, body: {stream_key: streamKey, terminal: true}})
					break
				}
			}
			
			console.log("Quest completed!")
		}
		fn()
	}
}
```
</details>

IF The Above Code Dosent Work try the Below Code

<details>
	<summary>Click to Expand</summary>

 
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
</details>
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

Credit Goes to Aamiaa and RussDev07

