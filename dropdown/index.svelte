<script>
  import { onMount, beforeUpdate } from 'svelte';
  import slugger from 'slugger';
  export let label;
  export let options;
  export let currentValue; // can be passed in as a prop or not. if not, the value will be undefined and markup below will use first option instead
  export let itemOnClick;
  export let subscribeTo;
  export let showLabel = true;
  export let isInline;
  export let isOrange;

  //to do: pass in an onsubscribe 

  $: currentValue = currentValue || options[0].value;
  $: activeDescendantID = 'dropdown-item-' + slugger(currentValue);
  $: activeDescendant = document.querySelector('#' + activeDescendantID);
  $: currentDisplay = options.find(d => d.value == currentValue) ? options.find(d => d.value == currentValue).display : options[0].display;
  itemOnClick = itemOnClick || function(){
    console.log('no itemOnClick prop');
    return;
  };
  
  if ( subscribeTo ){
    subscribeTo.subscribe(value => {
        currentValue = value || currentValue;
    });
  }
  let isOpen = false;
  let toBeSelected;
  function closeDropdown(){
    isOpen = false;
    toBeSelected = activeDescendant;
  }
  function clickHandler(e){
    
    if ( !isOpen ){
        isOpen = true;
        document.body.addEventListener('click', closeDropdown);
    } else {
        closeDropdown();
        document.body.removeEventListener('click', closeDropdown);
    }
  }

  function keyHandler(e){
    if ( e.keyCode !== 9 ){ // tab key. retain default for moving focus
        e.preventDefault();
    }
    console.log(e.keyCode);
    function dispatchClick(){
        toBeSelected.dispatchEvent(new MouseEvent('click'), {
            view: window,
            bubbles: true,
            cancelable: true
        });
    }
    if ( [32,38,40].indexOf(e.keyCode) !== -1 ) { // space, up arrow, down arrow
        if ( !isOpen ){
            isOpen = true;
        } else if ( e.keyCode === 32 ){ // space
            dispatchClick();
        } else if ( e.keyCode === 38 ){ // up
            let previousSibling = toBeSelected.previousElementSibling;
            if ( previousSibling ){
                toBeSelected = previousSibling;
            }
        } else {                        // down
            let nextSibling = toBeSelected.nextElementSibling;
            console.log(nextSibling);
            if ( nextSibling ){
                toBeSelected = nextSibling;
            }
        }
    }
    if ( e.keyCode === 27 && isOpen ){ // escape key
        closeDropdown();
    }
    if ( e.keyCode === 13 && isOpen ){ //enter
        dispatchClick();
    }
  }
  function itemClickHandler(e){
    console.log(this, this.dataset.value);
    currentValue = this.dataset.value;
    itemOnClick.call(this, e);
    isOpen = false;
  }
  function setDataAttributes(node, options){
      console.log(options);
      Object.keys(options).forEach(key => {
        node.dataset[key] = options[key];
      });
  }
  onMount(() => {
    activeDescendant = document.querySelector('#' + activeDescendantID);
    toBeSelected = activeDescendant;
  });
</script>

<style lang="scss">
    @import './../variables.scss';
    .dropdown-outer {
        width: 100%;
        font-size: 16px;
        font-weight: 700;
        color: $dark_blue;
        margin-right: 40px;
        margin-bottom: 5px;
        label {
            font-size: 16px;
            margin: 0;
            display: block;
        }
        &.isInline {
            width: auto;
            margin-right: 0;
        }
    }
    .dropdown-inner {
        position: relative;
        background-color: rgba(255,255,255,0);
        cursor: pointer;
        border-bottom: 4px solid #296ec3;
        background: url("data:image/svg+xml;charset=utf-8,%3Csvg class='local-navigation__link-chevron' xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='%23296ec3' d='M7.406 7.828L12 12.422l4.594-4.594L18 9.234l-6 6-6-6z'/%3E%3C/svg%3E") 100%/22px 22px no-repeat;
        height: 40px;
        color: #296ec3;
        .isOrange & {
            border-bottom: 4px solid #f47d4b;
            background: url("data:image/svg+xml;charset=utf-8,%3Csvg class='local-navigation__link-chevron' xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='%23f47d4b' d='M7.406 7.828L12 12.422l4.594-4.594L18 9.234l-6 6-6-6z'/%3E%3C/svg%3E") 100%/22px 22px no-repeat;
            color: #f47d4b;

        }
    }
    .dropdown {
        padding: 12px;
        font-size: 12px;
        font-weight: 700;
        background-color: rgba(255,255,255,0);
        & > div {
            text-transform: uppercase;
            position: relative;
            background-color: rgba(255,255,255,0);
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
            padding-right: 1em;
            .isInline & {
                text-overflow: initial;
            }
        }
        & > ul {
            position: absolute;
            display:none;
            top: 0;
            left: 0;
            min-width: 100%;
            background-color: #fff;
            z-index: 1;
            padding: 0;
            margin-left: 0;
            border: 1px solid #aeadad;
            height: 442px;
            max-height: calc(100vh - 100px);
            overflow-y: scroll;
            li {
                list-style-type: none;
                margin: 0;
                padding: 12px;
                font-size: 13px;
                &:hover, &.hover {
                    background-color: #eef6ff;
                    text-decoration: underline;
                    
                }
                &[aria-selected="true"]{
                    background-color: $blue;
                    color: #fff;
                }

            }
        }
        &.is-open {
            background-color: #296ec3;
            color: #fff;
        }
        &.is-open > ul {
            display: block;
            color: #296ec3;
        }
    }
</style>
<svelte:options accessors={true}/>
<div class:isOrange class:isInline class="dropdown-outer">
    <label class:visually-hidden="{!showLabel}">{label}</label>
    <div class="dropdown-inner">
        <div on:keydown="{keyHandler}" on:click|stopPropagation="{clickHandler}" class:is-open="{isOpen}" class="dropdown" aria-haspopup="listbox" aria-expanded={isOpen} role="button" tabindex="0">
            <div>{currentDisplay}</div>
            <ul role="listbox" aria-activedescendant="{activeDescendantID}">
            {#each options as option}
                <li use:setDataAttributes="{option}" class:hover="{toBeSelected ? option.value === toBeSelected.dataset.value : false}" on:click|stopPropagation="{itemClickHandler}" data-value="{option.value}" aria-selected="{currentValue === option.value}" role="option" id="dropdown-item-{slugger(option.value)}">{option.display}</li>
            {/each}
            </ul>
        </div>
    </div>
</div>