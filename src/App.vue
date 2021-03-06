<script>
import moment from 'moment';
import debounce from 'debounce';
import Note from '@/components/Note.vue';
import NoteEdit from '@/views/NoteEdit.vue';

class NoteClass {
   constructor() {
      this.name = 'New Note';
      this.content = '';
      this.creationDate = moment(new Date()).format('MMMM Do YYYY, h:mm:ss a');
   }
}

export default {
   name: 'app',

   // Components: Component registry to use
   components: { Note, NoteEdit },

   // Data: It is the container of variables, which may be accessible from the view => data: {}
   // When working with components, it changes to the function format => data(){}
   data: () => ({
      navOpen: false,
      notes: [],
      darkMode: true,
      interval: '',
      saved: true,
      search: '',
      selected: 0,
   }),

   // Properties calculated based on established logic. They are declared as functions.
   computed: {
      filteredNotes() {
         if (this.search) {
            return this.notes.filter((note) => {
               return note.name
                  .toLowerCase()
                  .includes(this.search.toLowerCase());
            });
         }

         return this.notes;
      },
   },

   // It is invoked after creating the instance
   created() {
      const notes = JSON.parse(localStorage.getItem('simple-notes'));
      if (notes) {
         this.notes = notes;
      } else {
         localStorage.setItem('simple-notes', JSON.stringify([]));
      }
   },

   /*
   It is invoked after an instance has been destroyed. All directives for VUE instances
   have been unbound, listeners have been removed.
   */
   destroyed() {
      clearInterval(this.interval);
   },

   // Methods: It allows to declare methods that could be used in our project
   methods: {
      newNote() {
         this.notes.unshift(new NoteClass());
         this.save();
      },

      selectNote(index) {
         this.selected = index;
         this.navOpen = false;
      },

      editNote(key, e) {
         this.notes[this.selected][key] = e.target.value;
         this.saved = false;
         this.debounceSave();
      },

      save() {
         localStorage.setItem('simple-notes', JSON.stringify(this.notes));
         this.saved = true;
      },

      debounceSave: debounce(function () {
         if (!this.saved) {
            this.save();
         }
      }, 3 * 1000),

      deleteNote(e, index) {
         e.stopPropagation();
         const confirmed = window.confirm('Delete note?');
         if (confirmed) {
            const newNotes = [...this.notes];
            newNotes.splice(index, 1);
            this.notes = newNotes;
            this.save();
         }
      },
   },
};
</script>

<template>
   <div :class="['app', { 'app--light-mode': !darkMode }]">
      <header class="header">
         <div :class="['menu', { navOpen }]" @click="navOpen = !navOpen">
            &#10095;
         </div>
         <div :class="['title', { 'title--light-mode': !darkMode }]">
            Simple Notes
         </div>
         <div class="last-saved-and-toggle">
            <div class="last-saved val">{{ saved ? 'Saved' : 'Unsaved' }}</div>
            <button
               :class="[
                  'theme-toggle',
                  { 'theme-toggle--light-mode': !darkMode },
               ]"
               @keydown.enter="darkMode = !darkMode"
               @click="darkMode = !darkMode"
            >
               {{ darkMode ? 'Light Mode' : 'Dark Mode' }}
            </button>
            <div class="mobile-theme-toggle">
               <div v-if="darkMode" @click="darkMode = !darkMode">
                  <img
                     class="mobile-theme-toggle__icon"
                     src="@/assets/sun.png"
                  />
               </div>
               <div v-else @click="darkMode = !darkMode">
                  <img
                     class="mobile-theme-toggle__icon"
                     src="@/assets/crescent-moon.png"
                  />
               </div>
            </div>
         </div>
      </header>

      <div class="body">
         <section :class="['nav', { navOpen, 'nav--light-mode': !darkMode }]">
            <div class="search">
               <input
                  :class="{ 'light-mode': !darkMode }"
                  placeholder="Search"
                  v-model="search"
               />
               <div
                  :class="['new-note', { 'new-note--light-mode': !darkMode }]"
                  @click="newNote"
               >
                  +
               </div>
            </div>

            <div class="notes">
               <Note
                  v-for="(note, i) in filteredNotes"
                  :key="i"
                  :note="note"
                  :selected="selected === i"
                  @click="selectNote(i)"
                  @delete="(e) => deleteNote(e, i)"
               />
            </div>
         </section>

         <section class="window">
            <NoteEdit
               :dark-mode="darkMode"
               :note="notes[selected]"
               @name="(e) => editNote('name', e)"
               @content="(e) => editNote('content', e)"
            />
         </section>
      </div>
   </div>
</template>

<style lang="scss">
$dark: #323231;
$darker: #212121;
$white: #f5f5f5;

.app {
   height: 100vh;
   max-height: 100vh;
   width: 100%;
   background: $darker;
   overflow: hidden;
   font-family: 'Avenir', Helvetica, Arial, sans-serif;
   -webkit-font-smoothing: antialiased;
   -moz-osx-font-smoothing: grayscale;
   text-align: center;
   color: $white;

   &--light-mode {
      background: $white;
      color: $dark;
   }
}

.header {
   height: 40px;
   width: 100%;
   display: flex;
   justify-content: space-between;
   align-items: center;
   padding: 0px 30px;
   position: relative;
   z-index: 10;
   box-shadow: 0px 3px 5px rgba(0, 0, 0, 0.5);

   div {
      flex: 1;
   }

   .menu {
      display: none;
   }

   .title {
      display: flex;
      justify-content: flex-start;
   }

   .last-saved {
      display: flex;
      justify-content: flex-end;
      color: #888;
      font-size: 12px;
   }

   .last-saved-and-toggle {
      align-items: center;
      display: flex;
   }

   .theme-toggle {
      background: none;
      border: 1px solid $white;
      border-radius: 3px;
      color: $white;
      cursor: pointer;
      font-weight: 600;
      margin-left: 16px;

      &--light-mode {
         border: 1px solid $dark;
         color: $dark;
      }

      @media (max-width: 750px) {
         display: none;
      }
   }

   .mobile-theme-toggle {
      @media (min-width: 750px) {
         display: none;
      }

      &__icon {
         background-position: center;
         background-repeat: no-repeat;
         background-size: cover;
         height: 20px;
         margin: 0 auto;
         width: 20px;
      }
   }

   @media (max-width: 750px) {
      .menu {
         display: flex;
         flex: none;
         justify-content: flex-start;
         align-items: center;

         &.navOpen {
            justify-content: flex-end;
            transform: rotate(180deg);
         }
      }

      .title {
         justify-content: center;
      }
   }
}

.body {
   height: calc(100% - 40px);
   width: 100%;
   display: flex;
   position: relative;

   .nav {
      height: 100%;
      width: 240px;
      background: $darker;
      padding-top: 10px;
      padding-bottom: 40px;
      position: relative;
      z-index: 9;
      box-shadow: 3px 0px 5px rgba(0, 0, 0, 0.25);

      &--light-mode {
         background: $white;
      }

      .search {
         height: 26px;
         width: 100%;
         display: flex;
         justify-content: space-between;
         margin-bottom: 10px;
         align-items: center;

         input {
            height: 100%;
            width: calc(100% - 32px);
            background: $dark;
            color: $white;
            border: 1px solid $dark;
            padding: 0px 6px;
            outline: none;
            &:focus {
               border: 1px solid #888888;
            }
         }

         .light-mode {
            background: $white;
            color: $dark;
         }

         .new-note {
            height: 26px;
            width: 32px;
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            transition: 0.1s background ease-in-out;
            &:hover {
               background: rgba(255, 255, 255, 0.25);
            }
            &:active {
               background: rgba(255, 255, 255, 0.12);
            }

            &--light-mode {
               &:hover {
                  background: rgba(250, 165, 0, 0.5);
               }

               &:active {
                  background: rgba(250, 165, 0, 0.21);
               }
            }
         }
      }

      .notes {
         height: 100%;
         width: 100%;
         border-top: 1px solid $dark;
         overflow-y: scroll;
         &::-webkit-scrollbar {
            width: 3px;
         }
         &::-webkit-scrollbar-track {
            background: transparent;
         }
         &::-webkit-scrollbar-thumb {
            background: #888;
         }
      }

      @media (max-width: 750px) {
         width: 100%;
         position: absolute;
         left: -100%;
         transition: 0.15s left ease-in-out;
         &.navOpen {
            left: 0;
         }
      }
   }

   .window {
      height: 100%;
      padding-bottom: 2rem;
      width: calc(100% - 240px);

      @media (max-width: 750px) {
         width: 100%;
      }
   }
}

.light-mode {
   background: $white;
   color: $dark;
}
</style>
