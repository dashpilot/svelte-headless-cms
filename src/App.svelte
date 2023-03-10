<script>
  import { onMount } from 'svelte';
  import { tick } from 'svelte';
  import {slugify} from './lib/Helpers.svelte';
  
  
  import Editor from './lib/Editor.svelte'
  
  let data;
  let loading = true;
  let saving = false;
  let saved = false;
  
  let curCatSlug = "home";
  let curTab = false;
  let curPostId;

  onMount(async () => {
    const res = await fetch(cfg.dataPath);
    data = await res.json();
    loading = false;
 
    await tick();
    if(document.querySelector('#post-list').firstChild){
      document.querySelector('#post-list').firstChild.click();
    }
  });
  
  async function setCat(slug){
    curCatSlug = slug;
    curTab = false;
    await tick();
    if(document.querySelector('#post-list').firstChild){
      document.querySelector('#post-list').firstChild.click();
    }
  }
  
  function truncateString(str, no_words) {
    str = str.replace(/(<([^>]+)>)/gi, "");
    return str.split(" ").splice(0,no_words).join(" ");
  }
  
  function newId(){
    let highest_id = Math.max(...data.posts.map(x => x.id));
    return highest_id+1;
  }
  
  function addPost(){
    
    let curCat = data.categories.filter(x=>x.slug==curCatSlug)[0];
    let curType = data.types.filter(x=>x.slug==curCat.type)[0];
  
    let newPost = {}
    let id = newId();
    
    newPost.title = "Lorem ipsum";
    newPost.id = id;
    newPost.category = curCatSlug;
    
    curType.fields.forEach(item=>{
      if(item.type=='gal'){
        newPost[item.title] = [];
      }else{
        newPost[item.title] = "";
      }
    })
    
    newPost.slug = slugify(newPost.title, id);
  
    data.posts.unshift(newPost)
    data = data;
    
    curPostId = newPost.id;
  }
  
  function save(){
    saving = true;
    let opts = {};
    opts.path = "data.json";
    opts.type = 'json';
    opts.data = data;
    call_api('api/save', opts).then(function(res) {
      if (res.status=='ok') {
        saving = false;
        saved = true;
        console.log('Saved');
        setTimeout(()=>{
          saved = false;
        }, 2000)
       
      } else {
        saving = false;
        console.log('Error saving');
      }
    });
  }
  
  function setTab(tab){
    curCatSlug = false;
    curTab = tab;
  }

  
</script>

{#if loading}

Loading
{:else}

<div class="row g-0">
  
  
  <!-- col 1 -->
  <div class="col-2 col1">
    
  <div class="settings">
    <!-- <div class="site-title">Dashpilot</div>-->
  </div>
    
    {#each data.categories as item}
    <a class="tab" class:active="{curCatSlug === item.slug}"
    on:click="{() => setCat(item.slug)}">{item.title}</a>
    {/each}
    
       
       <br />
 
    <div class="divider">
      <div class="site-title">Manage</div>
    </div>

    
  
    <a class="tab" class:active="{curTab === 'settings'}"
    on:click="{() => setTab('settings')}">Settings</a>
  
 
  </div>

  {#if curCatSlug}
  
  <!-- col 2 -->
  <div class="col-3 col2">
    <div class="filter">
      
      <div class="btn-group">
        <button class="btn btn-outline-dark" on:click={addPost}><i class="fas fa-plus"></i></button>
        
      </div>
      
    </div>
    
    <ul class="list-group" id="post-list">
      {#each data.posts.filter(x=>x.category==curCatSlug) as item}
      <li class="list-group-item" class:active2="{curPostId === item.id}"
      on:click={()=>curPostId=item.id}>
        
        <b>{item.title}</b>
        <br>
        <span>{truncateString(item.body, 7)}...</span>
        
      </li>
      {/each}
      
      
     
    
    </ul>
    
 
    
  </div>

  
  <!-- col 3 -->
  <div class="col-7 col3">
    
    <div class="savebar">
      <div class="buttons">
        <button class="btn btn-primary mt-1" on:click={save}>
          {#if saving}
          <i class="fas fa-spinner fa-spin"></i>
          {:else if saved}
          <i class="fas fa-check"></i>
          {:else}
          <i class="fas fa-save"></i>
          {/if}
          
         &nbsp; Save</button>
      </div>
      
    </div>
    <div class="post-editor">
  
    {#if curPostId}
     {#key curPostId}
     
    
      <Editor bind:curPostId bind:data />
     {/key}
    {/if}
    
    </div>

  </div>
  {:else}
  
  <div class="col10">
    
    <div class="savebar">
      
      <h4 class="float-start">Settings</h4>
      
      
      <div class="buttons">
        <button class="btn btn-primary mt-1" on:click={save}>
          {#if saving}
          <i class="fas fa-spinner fa-spin"></i>
          {:else if saved}
          <i class="fas fa-check"></i>
          {:else}
          <i class="fas fa-save"></i>
          {/if}
          
         &nbsp; Save</button>
      </div>
      
    </div>
    <div class="settings-editor">
      
      <div class="settings-inner">
      {#each Object.keys(data.settings) as key, val}
      <label>{key.replaceAll('_', ' ')}</label>
      <input type="text" class="form-control" bind:value={data.settings[key]} />
      {/each}
      </div>
 
    </div>
  
  </div>
  
  
  {/if}
  
  
</div>
{/if}