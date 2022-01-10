<script>
  // Basic Config
  let compendiumTypes = [
    "Actor",
    "Card",
    "Item",
    "JournelEntry",
    "Macro",
    "Playlist",
    "RollTable",
    "Scene",
  ];
  let selected = "Actor";
  let siteAddr = "https://fgen.mxzf.info/module/manifest/";

  let packs = [{ name: "", label: "", type: "Actor" }];

  let URL = "";
  let copy = "";

  function onSubmit(e) {
    copy = "";
    const formData = new FormData(e.target);

    const data = {};
    for (let field of formData) {
      const [key, value] = field;
      if (key == "title") {
        data["name"] = string_to_slug(value);
      }
      data[key] = value;
    }

    data.packs = [];
    for (let pair of packs) {
      let pack_data = {};
      for (let data of Object.keys(pair)) {
        if (data == "label") {
          pack_data["name"] = string_to_slug(pair.label);
        }
        pack_data[data] = pair[data];
      }
      data.packs.push(pack_data);
    }
    //data.packs = packs;
    console.log(data);
    URL = siteAddr + btoa(JSON.stringify(data)) + "/manifest.json";
    navigator.clipboard.writeText(URL);
    copy = "Copied to clipboard";
    //'http://localhost:8000/module/manifest/'+  btoa(JSON.stringify(data)) + '/manifest.json'
  }

  function string_to_slug(str) {
    str = str.replace(/^\s+|\s+$/g, ""); // trim
    str = str.toLowerCase();

    // remove accents, swap ñ for n, etc
    var from = "àáäâèéëêìíïîòóöôùúüûñç·/_,:;";
    var to = "aaaaeeeeiiiioooouuuunc------";
    for (var i = 0, l = from.length; i < l; i++) {
      str = str.replace(new RegExp(from.charAt(i), "g"), to.charAt(i));
    }

    str = str
      .replace(/[^a-z0-9 -]/g, "") // remove invalid chars
      .replace(/\s+/g, "-") // collapse whitespace and replace by -
      .replace(/-+/g, "-"); // collapse dashes

    return str;
  }

  function add(e) {
    e.preventDefault();
    packs = packs.concat({ name: "", label: "", type: "Actor" });
    URL = "";
    copy = "";
  }

  function clear(e) {
    e.preventDefault();
    packs = [{ name: "", label: "", type: "Actor" }];
    URL = "";
    copy = "";
  }

  // {'name': "test_module", 'title': 'Test Module'
</script>

<main>
  <form on:submit|preventDefault={onSubmit}>
    <fieldset>
      <legend>Module Properties</legend>
      <div class="flex">
        <label for="title">Module Name</label>
        <input type="text" id="title" name="title" value="" />
      </div>
    </fieldset>
    <fieldset>
      <legend>Packs</legend>
      <div class="row">
        {#each packs as pack}
          <div class="col col-1-2">
            <fieldset>
              <div class="flex">
                <label for="label" class="center">Name</label>
                <input type="text" bind:value={pack.label} />
              </div>
              <div  class="flex">
                <label for="Type" class="center">Type</label>

                <select bind:value={pack.type}>
                  {#each compendiumTypes as compendium}
                    <option value={compendium}>{compendium}</option>
                  {/each}
                </select>
              </div>
            </fieldset>
          </div>
        {/each}
      </div>
    </fieldset>

    <button on:click={add}> Add Pack </button>

    <button on:click={clear} class="button secondary"> Clear Packs </button>
    <button type="submit">Generate Module</button>
  </form>

  <div>
    <label for="name">URL:</label>
    <input type="text" bind:value={URL} />
  </div>
  <div class="green">{copy}</div>
</main>

<style>
  main {
    max-width: 680px;
    margin: 100px auto 20px;
    padding: 0 10px;
  }

  legend {
    font-size: 24px;
    font-weight: 700;
    margin: 0 0 8px;
    color: #fff;
  }

  .green {
    color: green;
    text-align: center;
  }

  .flex { 
      display: flex;
  }

  .flex label {
      width: 100px;
  }

  .center { 
      margin-top: 12px;
      margin-right: 5px;
      text-align: right;
    
  }
</style>
