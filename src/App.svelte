<script>
  import { onMount } from 'svelte';
  import { tick } from 'svelte';
  
  import Editor from './lib/Editor.svelte'
  
  let data;
  let loading = true;
  let curCatSlug = "home";
  
  let curPostId;

  onMount(async () => {
    const res = await fetch(cfg.dataPath);
    data = await res.json();
    loading = false;
  });
  
  async function setCat(slug){
    curCatSlug = slug;
    await tick();
    if(document.querySelector('#post-list').firstChild){
      document.querySelector('#post-list').firstChild.click();
    }
  }
  
  function truncateString(str, no_words) {
    return str.split(" ").splice(0,no_words).join(" ");
  }
  
  function addPost(){
    
    let curCat = data.categories.filter(x=>x.slug==curCatSlug)[0];
    let curType = data.types.filter(x=>x.slug==curCat.type)[0];
  
    let newPost = {}
    newPost.title = "Lorem ipsum";
    newPost.id = "i"+Date.now();
    newPost.slug = "";
    newPost.category = curCatSlug;
    
    curType.fields.forEach(item=>{
      if(item.type=='gal'){
        newPost[item.title] = [];
      }else{
        newPost[item.title] = "";
      }
    })
  
    data.posts.unshift(newPost)
    data = data;
    
    curPostId = newPost.id;
  }
  
  function save(){
    let opts = {};
    opts.path = "data.json";
    opts.type = 'json';
    opts.data = data;
    call_api('api/save', opts).then(function(res) {
      if (res.ok) {
        console.log('Saved');
      } else {
        console.log('Error saving');
      }
    });
  }

  
</script>

{#if loading}

Loading
{:else}

<div class="row g-0">
  
  
  <!-- col 1 -->
  <div class="col-2 col1">
    
  <div class="settings">
    <div class="site-title">Dashpilot</div>
  </div>
    
    {#each data.categories as item}
    <a class="tab" class:active="{curCatSlug === item.slug}"
    on:click="{() => setCat(item.slug)}">{item.title}</a>
    {/each}
 
  </div>

  
  
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
        <span>{@html truncateString(item.body, 7)}...</span>
        
      </li>
      {/each}
    
    </ul>
    
  </div>

  
  
  
  <!-- col 3 -->
  <div class="col-7 col3">
    
    <div class="savebar">
      <div class="buttons">
        <button class="btn btn-primary mt-1" on:click={save}>Save</button>
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
  
  
</div>
{/if}