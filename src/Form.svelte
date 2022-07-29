<script>
  import { onMount } from "svelte";
  import Svelecte from "svelecte";

  // Basic Config
  let compendiumTypes = [
    "Actor",
    "Adventure",
    "Cards",
    "Item",
    "JournalEntry",
    "Macro",
    "Playlist",
    "RollTable",
    "Scene",
  ];

  let selected = "Actor";
  let siteAddr =
    location.protocol.concat("//").concat(window.location.host) +
    "/module/manifest/";

  //remote source for search data
  const dataURL =
    location.protocol.concat("//").concat(window.location.host) +
    "/systems.json";
  let searchData = [];
  let options = {};
  let myValue = "";

  let packs = [{ name: "", label: "", type: "Actor" }];

  let URL = "";
  let copy = "";

  function onSubmit(e) {
    copy = "";
    const formData = new FormData(e.target);

    //Create json data from form field
    // {title:String, author:String, description:String}
    const data = {};
    for (let field of formData) {
      const [key, value] = field;
      if (key == "title") {
        data["name"] = string_to_slug(value);
      }
      data[key] = value;
    }

    //Create array from Packs object
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

    //Base64 the object to make the manifest URL
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

  // Pack Management GUI funnctions
  // e = click Event

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

  function delete_pack(e) {
    e.preventDefault();
    packs = packs.filter((m) => m != packs[e.target.id]);
    if (packs == "") {
      clear(e);
    }
  }

  //Override onMount to make it async. Use for remote data source for searching

  onMount(async function () {
    const response = await fetch(dataURL, options);
    searchData = await response.json();
    console.log(searchData);
  });
</script>

<main>
  <link
    rel="stylesheet"
    href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css"
  />
  <h1>Shared Compendium Creator</h1>
  <form on:submit|preventDefault={onSubmit}>
    <fieldset>
      <legend>Module Properties</legend>
      <div class="flex">
        <label for="title">Module Name</label>
        <input type="text" id="title" name="title" value="" />
      </div>
      <div class="flex">
        <label for="author" class="center">Author</label>
        <input type="text" id="author" name="author" value="" />
      </div>
      <div class="flex">
        <label for="description" class="center">Description</label>
        <input type="text" id="description" name="description" value="" />
      </div>
      <div class="flex">
        <label for="system" class="center">System</label>
        <Svelecte
          options={searchData.systems}
          bind:value={myValue}
          name="system"
          inputId="system"
          creatable=true
        />
      </div>
    </fieldset>
    <fieldset>
      <legend>Packs</legend>
      <div class="row">
        {#each packs as pack, _id}
          <div>
            <fieldset>
              <div class="flex center">
                <div class="flex">
                  <label for="label" class="center">Name</label>
                  <input type="text" bind:value={pack.label} />
                </div>
                <div class="flex">
                  <label for="Type" class="center">Type</label>

                  <select bind:value={pack.type}>
                    {#each compendiumTypes as compendium}
                      <option value={compendium}>{compendium}</option>
                    {/each}
                  </select>
                </div>
                <div class="flex">
                  {#if packs.length > 1}<i
                      on:click={delete_pack}
                      id={_id}
                      class="fa fa-trash-o">&nbsp;</i
                    >{/if}
                </div>
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
    <input type="text" bind:value={URL} readonly />
  </div>
  <div class="green">{copy}</div>
  <div>
    <h2>To Install:</h2>
  </div>
  <ul>
    <li>Open the Foundry Setup page</li>
    <li>Click on the Add-on Modules tab</li>
    <li>Click the Install Module button near the bottom</li>
    <li>
      Paste the manifest URL generated above into the field at the bottom of the
      dialog
    </li>
    <li>Click Install</li>
  </ul>
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
    margin-right: 5px;
    text-align: right;
  }

  .flex i {
    font-size: 24px;
    width: 24px;
    margin-left: 50px;
    margin-right: 5px;
    text-align: right;
    margin-top: 7px;
    color: #4d6deb;
  }

  .flex i:hover {
    color: red;
  }

  .center {
    margin-top: 12px;
  }

  li {
    line-height: 1.5;
  }

  :global(.svelecte-control) {
    --sv-bg: #37425f !important;
    --sv-active-border: 2px solid #4d6deb !important;
    --sv-border-color: #4d6deb !important;
    --sv-color: #fff !important;
    --sv-item-color: #fff !important;
    --sv-item-selected-bg: rgba(255, 255, 255, 0.4) !important;
    --sv-item-active-bg: rgba(255, 255, 255, 0.4) !important;
    --sv-highlight-bg: rgb(43, 43, 43) !important;
    font-size: 18px;
    width:100% !important;
  }

  :global(.sv-dropdown) {
    font-size: 18px;
    background: #37425f;
    color: #fff;
    width: 100%;
    border-radius: 6px;
    border: 2px solid #4d6deb;
    padding: 0 6px;
    margin: 0 0 16px;
    outline: none;
    box-sizing: border-box;
  }
</style>
