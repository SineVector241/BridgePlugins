<template>
    <div style="padding:10px; overflow:scroll;">
        <h1>JSON RawText Generator</h1>
        <div class="TitleButtonsRTGrid">
            <v-btn block color="primary" @click="AddMessageModule">Add Message</v-btn>
            <v-btn block color="primary" @click="AddScoreModule">Add Score</v-btn>
            <v-btn block color="primary" @click="AddTranslationModule">Add Translation</v-btn>
            <v-btn block color="primary" @click="AddSelectorModule">Add Selector</v-btn>
            <v-btn block>Add Text-Modifier</v-btn>
        </div>
        <v-btn style="margin-top: 5px;" color="primary" @click="Generate" :disabled="Modules.length == 0">Generate</v-btn>
        <v-btn style="margin-top: 5px;" @click="CopyOutput" :disabled="GeneratedRawtext == 'No Data'">Copy Output</v-btn>
        <br></br>
        <div class="Output">
            <p>{{ GeneratedRawtext }}</p>
        </div>
        <div class="RTEditor">
            <h2>Editor</h2>
            <transition-group name="RT-flip-list" tag="v-card">
            <v-card class="RTModule" v-for="(module, i) in Modules" style="margin-bottom: 0.8rem" color="sidebarSelection" :key="module.Id">
                <v-card-title>
                    {{ module.Type }}
                </v-card-title>
                <v-card-text>
                    <v-text-field v-if="module.Type == 'Message'" outlined label="Message" v-model="module.Message"></v-text-field>
                    <v-text-field v-if="module.Type == 'Score'" outlined label="Objective" v-model="module.Objective"></v-text-field>
                    <v-text-field v-if="module.Type == 'Score'" outlined label="Score Target(Selector/FakePlayer)" v-model="module.TargetName"></v-text-field>
                    <v-text-field v-if="module.Type == 'Translation'" outlined label="Translation Id" v-model="module.TranslationString"></v-text-field>
                    <v-text-field v-if="module.Type == 'Selector'" outlined label="Selector" v-model="module.Selector"></v-text-field>
                    <v-selection v-if="module.Type == 'Text Modifier'" label="Text Modifier" v-model="module.TextMod"></v-selection>
                </v-card-text>
                <v-card-actions>
                    <v-btn icon color="error" @click="DeleteModule(i)">
						<v-icon> mdi-delete </v-icon>
					</v-btn>
                    <v-btn icon color="primary" @click="MoveModuleUp(i)" :disabled="i == 0">
						<v-icon> mdi-arrow-up-box </v-icon>
					</v-btn>
                    <v-btn icon color="primary" @click="MoveModuleDown(i)" :disabled="i == Modules.length-1">
						<v-icon> mdi-arrow-down-box </v-icon>
					</v-btn>
                </v-card-actions>
            </v-card>
        </transition-group>
        </div>
        <div class="Toast" id="Popup">
            <h2>Copied</h2>
        </div>
    </div>
</template>

<script>
export default {
    data: () => ({
        Modules: [],
        GeneratedRawtext: "No Data"
    }),
    methods: {
        Generate() {
            var GeneratedModules = "";
            this.Modules.forEach(module => {
                GeneratedModules += `${module.Construct()},`;
            });
            GeneratedModules = GeneratedModules.slice(0,-1);
            this.GeneratedRawtext = `{"rawtext":[${GeneratedModules}]}`;
        },
        CopyOutput() {
            navigator.clipboard.writeText(this.GeneratedRawtext);
            var Toast = document.getElementById("Popup");
            Toast.style.animation = "PopupAnimation 2s";
            Toast.addEventListener("animationend", (ev) => {
                ev.target.style.animation = "";
                Toast.removeEventListener("animationend", (ev) => { });
            })
        },
        AddMessageModule() {
            this.Modules.push(new Message());
        },
        AddScoreModule() {
            this.Modules.push(new Score());
        },
        AddTranslationModule() {
            this.Modules.push(new Translation());
        },
        AddSelectorModule() {
            this.Modules.push(new Selector());
        },
        AddTextModificationModule() {
            this.Modules.push(new TextModification());
        },
        MoveModuleUp(Index) {
            MoveArray(this.Modules,Index, Index-1)
        },
        MoveModuleDown(Index) {
            MoveArray(this.Modules,Index, Index+1)
        },
        DeleteModule(Id) {
            this.Modules.splice(Id,1);
        }
    }
}

function MoveArray(arr, fromIndex, toIndex) {
    var element = arr[fromIndex];
    arr.splice(fromIndex, 1);
    arr.splice(toIndex, 0, element);
}

const TextMods = Object.freeze({
    NONE: "",
    BLACK: "§0",
    DARK_BLUE: "§1",
    DARK_GREEN: "§2",
    DARK_AQUA: "§3",
    DARK_RED: "§4",
    DARK_PURPLE: "§5",
    GOLD: "§6",
    GRAY: "§7",
    DARK_GRAY: "§8",
    BLUE: "§9",
    GREEN: "§a",
    AQUA: "§b",
    RED: "§d",
    LIGHT_PURPLE: "§d",
    YELLOW: "§e",
    WHITE: "§f",
    MINECOIN_GOLD: "§g",
    RESET: "§r",
    ITALICS: "§o",
    BOLD: "§l",
    OBFUSCATED: "§k"
});

const ModuleTypes = Object.freeze({
    MESSAGE: "Message",
    SCORE: "Score",
    TRANSLATION: "Translation",
    SELECTOR: "Selector",
    TEXT_MODIFICATION: "Text Modifier"
});

class Message {
    constructor() {
        this.Id = crypto.randomUUID(),
        this.Type = ModuleTypes.MESSAGE;
        this.Message = "";
    }

    Construct() {
        return `{"text":"${this.Message}"}`;
    }
}

class Score {
    constructor() {
        this.Id = crypto.randomUUID(),
        this.Type = ModuleTypes.SCORE;
        this.TargetName = "*";
        this.Objective = "";
    }

    Construct() {
        return `{"score":{"name":"${this.TargetName}","objective":"${this.Objective}"}}`;
    }
}

class Translation {
    constructor() {
        this.Id = crypto.randomUUID(),
        this.Type = ModuleTypes.TRANSLATION;
        this.TranslationString = "commands.op.success";
    }

    Construct() {
        return `{"translate":"${this.TranslationString}"}`;
    }
}

class Selector {
    constructor() {
        this.Id = crypto.randomUUID(),
        this.Type = ModuleTypes.SELECTOR;
        this.Selector = "@e[type=player]";
    }

    Construct() {
        return `{"selector":"${this.Selector}"}`;
    }
}

class TextModification {
    constructor() {
        this.Id = crypto.randomUUID(),
        this.Type = ModuleTypes.TEXT_MODIFICATION;
        this.TextMod1 = TextMods.NONE;
        this.TextMod2 = TextMods.NONE;
        this.TextMod3 = TextMods.NONE;
    }
}
</script>

<style>
.RTEditor {
    background: rgb(32,32,32);
    border-radius: 5px;
    padding: 5px;
    margin-top: 5px;
}

.TitleButtonsRTGrid {
  justify-content: space-evenly;
  margin: 0 auto;
  display: grid;
  grid-gap: 1rem;
}

@media (min-width: 400px) {
  .TitleButtonsRTGrid { grid-template-columns: repeat(2, 1fr); }
}

@media (min-width: 600px) {
  .TitleButtonsRTGrid { grid-template-columns: repeat(3, 1fr); }
}

@media (min-width: 900px) {
  .TitleButtonsRTGrid { grid-template-columns: repeat(4, 1fr); }
}

@media (min-width: 1200px) {
  .TitleButtonsRTGrid { grid-template-columns: repeat(5, 1fr); }
}

.RT-flip-list-move {
  transition: transform 0.4s;
}
.RT-flip-list-enter-active, .RT-flip-list-leave-active {
  transition: all 0.4s;
}
.RT-flip-list-enter, .RT-flip-list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}
</style>