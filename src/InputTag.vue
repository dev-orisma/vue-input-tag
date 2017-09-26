<script>
  /*eslint-disable*/
  const validators = {
    email: new RegExp(/^(([^<>()[\]\\.,;:\s@\"]+(\.[^<>()[\]\\.,;:\s@\"]+)*)|(\".+\"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/),
    url : new RegExp(/^(https?|ftp|rmtp|mms):\/\/(([A-Z0-9][A-Z0-9_-]*)(\.[A-Z0-9][A-Z0-9_-]*)+)(:(\d+))?\/?/i),
    text : new RegExp(/^[a-zA-Z]+$/),
    digits : new RegExp(/^[\d() \.\:\-\+#]+$/),
    isodate : new RegExp(/^\d{4}[\/\-](0?[1-9]|1[012])[\/\-](0?[1-9]|[12][0-9]|3[01])$/)
  }
  /*eslint-enable*/

  export default {
    name: 'InputTag',

    props: {
      tags: {
        type: Array,
        default: () => []
      },
      tagsUpdate: {
        type: String,
        default: ''
      },
      placeholder: {
        type: String,
        default: ''
      },
      onChange: {
        type: Function
      },
      onFetch: {
        type: Function
      },
      onInput: {
        type: Function
      },
      readOnly: {
        type: Boolean,
        default: false
      },
      validate: {
        type: String,
        default: ''
      }
    },

    data () {
      return {
        newTag: '',
        suggest: {},
        showSuggestTag: false,
        listLength: 0,
        listIndex: -1,
        listElement: {},
      }
    },

    methods: {
      focusNewTag () {
        if (this.readOnly) { return }
        this.$el.querySelector('.new-tag').focus()
      },

      addNew (tag) {
        if (tag && this.tags.indexOf(tag) === -1 && this.validateIfNeeded(tag)) {
          this.tags.push(tag)
          this.tagChange()
        }
        this.newTag = ''
        this.tagHide()
      },

      validateIfNeeded (tagValue) {
        if (this.validate === '' || this.validate === undefined) {
          return true
        } else if (Object.keys(validators).indexOf(this.validate) > -1) {
          return validators[this.validate].test(tagValue)
        }
        return true
      },

      remove (index) {
        this.tags.splice(index, 1)
        this.tagChange()
      },

      removeLastTag () {
        if (this.newTag) { return }
        this.tags.pop()
        this.tagChange()
      },

      tagChange () {
        if (this.onChange) {
          // avoid passing the observer
          this.onChange(JSON.parse(JSON.stringify(this.tags)))
        }
      },

      tagInput (e) {
      	var _this = this
      	if(e.key == "ArrowUp"){
      		if(this.listIndex <= 0){
      			this.listIndex = 0
      		}else{
      			this.listIndex = this.listIndex - 1
      		}
      		this.setActiveSuggest(this.listIndex)
      	}else if(e.key == "ArrowDown"){
      		if(this.listIndex == this.listLength){
      			this.listIndex = this.listLength
      		}else{
      			this.listIndex = this.listIndex + 1
      		}
      		this.setActiveSuggest(this.listIndex)
      	}else if(e.key == "ArrowRight"){

      	}else if(e.key == "ArrowLeft"){

      	}else{
	  		_this.tagHide()
	      	_this.suggest = {};
	  		this.onFetch(this.newTag,function(list){
	  			_this.listLength = list.length - 1
	  			_this.listIndex = -1
	  			if(list.length > 0){
	  				_this.suggest = list;
	  				_this.tagShow()
	  			}
	  		})
      	}
      },
		tagShow(){
			this.showSuggestTag = true
			 setTimeout(() => document.addEventListener('click',this.tagHide), 0);
		},
		tagHide(){
			this.showSuggestTag = false
			 document.removeEventListener('click',this.tagHide);
		},
		setActiveSuggest(index){
			var _this = this
			var elm = document.getElementsByClassName('suggest-tag-list-item');
			var count = 0
			Array.prototype.forEach.call(elm, function(el) {
			    if(count == index){
			    	el.className = 'suggest-tag-list-item active'
			    	_this.newTag = el.innerText
			    }else{
			    	el.className = 'suggest-tag-list-item'
			    }
			    count++
			});
		}
    }  }
</script>

<template>

  <div @click="focusNewTag()" v-bind:class="{'read-only': readOnly}" class="vue-input-tag-wrapper">
    <span v-for="(tag, index) in tags" v-bind:key="index" class="input-tag">
      <span>{{ tag }}</span>
      <a v-if="!readOnly" @click.prevent.stop="remove(index)" class="remove"></a>
    </span>
    <input v-if="!readOnly" v-bind:placeholder="placeholder" type="text" v-model="newTag" v-on:keydown.delete.stop="removeLastTag()" v-on:keydown.enter.tab.prevent.stop="addNew(newTag)" v-on:keyup="tagInput" class="new-tag"/>
    <div id="suggest-tag" v-if="showSuggestTag == true">
    	<ul id="suggest-tag-list">
    		<li class="suggest-tag-list-item" v-on:click="addNew(item_tag.title)" v-for="item_tag of suggest">
    			<span>{{item_tag.title}}</span>
    		</li>
    	</ul>
    </div>
  </div>

</template>

<style>
	#suggest-tag{
	    background-color: #fff;
	    border: 1px solid #E0E0E0;
	    padding: 6px 0;
	    border-radius: 2px;
	    box-sizing: border-box;
		padding: 0;
	    max-height: 250px;
	    overflow: scroll;
	    width: 100%;
	    position: absolute;
	    z-index: 2;
	    top: 39px;
	    left: 0px;
	}
	#suggest-tag #suggest-tag-list{
	    margin: 0;
		padding: 0;
		list-style: none;
	}
	#suggest-tag #suggest-tag-list	.suggest-tag-list-item{
		position: relative;
	    font-size: 14px;
	    line-height: 16px;
	    border-bottom: 1px solid rgba(136,138,163,.3);
	    box-sizing: border-box;
	    cursor: pointer;
	    outline: none;
    	font-size: 20px!important;
	    line-height: 28px!important;
	    padding: 5px 15px!important;
	}

	#suggest-tag #suggest-tag-list	.suggest-tag-list-item:hover{
		background: rgba(135,138,163,.1);
	}
	#suggest-tag #suggest-tag-list	.suggest-tag-list-item.active{
		background: rgba(135,138,163,.1);
	}
  .vue-input-tag-wrapper {
    background-color: #fff;
    border: 1px solid #ccc;
    overflow: hidden;
    padding-left: 4px;
    padding-top: 4px;
    cursor: text;
    text-align: left;
    -webkit-appearance: textfield;
  }

  .vue-input-tag-wrapper .input-tag {
    background-color: #cde69c;
    border-radius: 2px;
    border: 1px solid #a5d24a;
    color: #638421;
    display: inline-block;
    font-size: 13px;
    font-weight: 400;
    margin-bottom: 4px;
    margin-right: 4px;
    padding: 3px;
  }

  .vue-input-tag-wrapper .input-tag .remove {
    cursor: pointer;
    font-weight: bold;
    color: #638421;
  }

  .vue-input-tag-wrapper .input-tag .remove:hover {
    text-decoration: none;
  }

  .vue-input-tag-wrapper .input-tag .remove::before {
    content: " x";
  }

  .vue-input-tag-wrapper .new-tag {
    background: transparent;
    border: 0;
    color: #777;
    font-size: 13px;
    font-weight: 400;
    margin-bottom: 6px;
    margin-top: 1px;
    outline: none;
    padding: 4px;
    padding-left: 0;
    width: 80px;
  }

  .vue-input-tag-wrapper.read-only {
    cursor: default;
  }

</style>
