<template>
  <div class="relative inline-flex">
    <button
      ref="trigger"
      class="btn bg-white border-slate-200 hover:border-slate-300 text-slate-500 hover:text-slate-600"
      aria-haspopup="true"
      @click.prevent="dropdownOpen = !dropdownOpen"
      :aria-expanded="dropdownOpen"
    >
      <span class="sr-only">Filter</span><wbr />
      <svg class="w-4 h-4 fill-current" viewBox="0 0 16 16">
        <path d="M9 15H7a1 1 0 010-2h2a1 1 0 010 2zM11 11H5a1 1 0 010-2h6a1 1 0 010 2zM13 7H3a1 1 0 010-2h10a1 1 0 010 2zM15 3H1a1 1 0 010-2h14a1 1 0 010 2z" />
      </svg>
    </button>
    <transition
      enter-active-class="transition ease-out duration-200 transform"
      enter-from-class="opacity-0 -translate-y-2"
      enter-to-class="opacity-100 translate-y-0"
      leave-active-class="transition ease-out duration-200"
      leave-from-class="opacity-100"
      leave-to-class="opacity-0"
    >
      <div v-show="dropdownOpen" class="origin-top-right z-10 absolute top-full min-w-72 bg-white border border-slate-200 pt-1.5 rounded shadow-lg overflow-hidden mt-1" :class="align === 'right' ? 'right-0' : 'left-0'">
        <div ref="dropdown">
          <div class="text-xs font-semibold text-slate-400 uppercase pt-1.5 pb-2 px-4">Filters</div>
          <div class="md:flex space-y-4 md:space-y-0 md:space-x-4">
            <div class="flex-1">
              <ul class="mb-4">
                <li class="py-1 px-3">
                  <label class="flex items-center">
                    <input type="checkbox" class="form-checkbox" data-sort="rewarded" checked/>
                    <span class="text-sm font-medium ml-2">Rewarded</span>
                  </label>
                </li>
                <li class="py-1 px-3">
                  <label class="flex items-center">
                    <input type="checkbox" class="form-checkbox" data-sort="no_reward" checked/>
                    <span class="text-sm font-medium ml-2">No Reward</span>
                  </label>
                </li>
                <li class="py-1 px-3">
                  <label class="flex items-center">
                    <input type="checkbox" class="form-checkbox" data-sort="rejected" checked/>
                    <span class="text-sm font-medium ml-2">Rejected</span>
                  </label>
                </li>
                <li class="py-1 px-3">
                  <label class="flex items-center">
                    <input type="checkbox" class="form-checkbox" data-sort="pending" checked/>
                    <span class="text-sm font-medium ml-2">Pending</span>
                  </label>
                </li>
                <li class="py-1 px-3">
                  <label class="flex items-center">
                    <input type="checkbox" class="form-checkbox" data-sort="validation" checked/>
                    <span class="text-sm font-medium ml-2">In Validation</span>
                  </label>
                </li>
              </ul>
            </div>
            <div class="flex-1">
              <ul class="mb-4">
                <li class="py-1 px-3">
                  <label class="flex items-center">
                    <input type="checkbox" class="form-checkbox" data-sort="declined" checked/>
                    <span class="text-sm font-medium ml-2">Declined</span>
                  </label>
                </li>
                <li class="py-1 px-3">
                  <label class="flex items-center">
                    <input type="checkbox" class="form-checkbox" data-sort="bonus" checked/>
                    <span class="text-sm font-medium ml-2">Bonus</span>
                  </label>
                </li>
                <li class="py-1 px-3">
                  <label class="flex items-center">
                    <input type="checkbox" class="form-checkbox" data-sort="load" checked/>
                    <span class="text-sm font-medium ml-2">Load</span>
                  </label>
                </li>
                <li class="py-1 px-3">
                  <label class="flex items-center">
                    <input type="checkbox" class="form-checkbox" data-sort="refund" checked/>
                    <span class="text-sm font-medium ml-2">Refunded</span>
                  </label>
                </li>
                <li class="py-1 px-3">
                  <label class="flex items-center">
                    <input type="checkbox" class="form-checkbox" data-sort="completed" checked/>
                    <span class="text-sm font-medium ml-2">Completed</span>
                  </label>
                </li>
              </ul>
            </div>
          </div>
          <div class="py-2 px-3 border-t border-slate-200 bg-slate-50">
            <ul class="flex items-center justify-between">
              <li>
                <button class="btn-xs bg-white border-slate-200 hover:border-slate-300 text-slate-500 hover:text-slate-600" @click="clean">Clear</button>
              </li>
              <li>
                <button class="btn-xs bg-indigo-500 hover:bg-indigo-600 text-white" @click="apply" @focusout="dropdownOpen = false">Apply</button>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import {computed, onMounted, onUnmounted, ref, watch} from 'vue'

export default {
  name: 'DropdownFilter',
  props: ['align', 'sorter'],
  emits: ['apply-sort'],
  setup(props,context) {

    const defaultSorter = ref(new Set());
    const sorter = computed( () => props.sorter);
    const tempChecked = ref(new Set());
    const dropdownOpen = ref(false);
    const trigger = ref(null);
    const dropdown = ref(null);

    watch(sorter, (data) => {
      defaultSorter.value = data;
    });

    // close on click outside
    const clickHandler = ({ target }) => {
      if (!dropdownOpen.value || dropdown.value.contains(target)) return;
      if(trigger.value.contains(target)){
        //console.log("sorterValue",sorter.value);
        document.querySelectorAll('input[type=checkbox]').forEach(value => {
          if(defaultSorter.value.has(value.dataset.sort)){
            value.checked = true
          }
        })
        return;
      }
      dropdownOpen.value = false;
    }

    // close if the esc key is pressed
    const keyHandler = ({ keyCode }) => {
      if (!dropdownOpen.value || keyCode !== 27) return
      dropdownOpen.value = false;
    }

    onMounted(() => {
      defaultSorter.value = new Set([...document.querySelectorAll('input[type=checkbox]')].map(formatValue => formatValue.dataset.sort));
      //sorter.value = defaultSorter.value;

      document.addEventListener('click', clickHandler)
      document.addEventListener('keydown', keyHandler)
    })

    onUnmounted(() => {
      document.removeEventListener('click', clickHandler)
      document.removeEventListener('keydown', keyHandler)
    })

    const clean = () => {
      const checked = document.querySelectorAll('input[type=checkbox]:checked');
      //tempChecked.value = checked;
      checked.forEach(value => value.checked = false);
    };

    const apply = () => {
      let checked = document.querySelectorAll('input[type=checkbox]:checked');
      checked = new Set([...checked].map(formatValue => formatValue.dataset.sort));
      tempChecked.value = checked;
      dropdownOpen.value = false;
      context.emit('apply-sort', checked);
    }

    return {
      dropdownOpen,
      trigger,
      dropdown,
      clean,
      apply
    }
  }
}
</script>
