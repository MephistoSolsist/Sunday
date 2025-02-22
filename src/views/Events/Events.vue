<template>
  <div class="flex">
    <div class="flex flex-col h-full w-full items-center sm:(w-[24vw] min-w-[270px] border-r border-gray-400/30)">
      <div
        class="bg-gray-50 border-t border-base-standout/70 flex flex-col order-last w-full px-0.5 self-end items-center sm:(border-t-0 order-first border-b px-0 pb-0.5)">
        <input
          type="text"
          class="border-base-standout rounded-lg order-last h-10 shadow-inner my-0.5 mx-0.5 text-center sm:(rounded text-left)"
          style="width: 98%"
          v-model="searchQuery"
          placeholder="搜索" />
        <TabGroup class="w-full" :defaultIndex="0">
          <TabList class="flex space-x-1 p-1">
            <Tab v-for="item in roleFilter" as="template" :key="item.name" v-slot="{ selected }">
              <button
                @click="filterHandler(item)"
                class="rounded-lg font-semibold w-full py-2.5 text-indigo-600 leading-5 focus:(outline-none border-base-standout )"
                :class="[
                  selected
                    ? 'bg-white shadow cursor-default'
                    : 'text-gray-400 hover:bg-gray-50/[0.12] hover:text-blue-400',
                ]">
                {{ item.name }}
              </button>
            </Tab>
          </TabList>
        </TabGroup>
      </div>
      <scroll-area @scroll="onScroll">
        <div class="hidden sm:block">
          <button
            v-for="item in filteredList"
            :key="item.eventId"
            class="flex flex-row flex-nowrap cell justify-between"
            :class="[item.eventId == eventStore.eventId ? 'bg-gray-400/30 cursor-default shadow' : '']"
            @click="showDetail(item)">
            <div class="text-left w-2/3 truncate">
              {{ item.problem }}
            </div>
            <div class="uppercase">
              {{ item.status }}
            </div>
          </button>
        </div>
        <!-- mobile -->
        <div class="sm:hidden flex flex-col-reverse">
          <event-card
            v-for="item in filteredList"
            :key="item.eventId"
            :class="isCurrentMember(item, memberId) ? 'h-30' : ''"
            :bannerMessage="item.status == 'committed' && eventsMatchingByRID ? '已提交' : ''">
            <template #body>
              {{ item.problem }}
            </template>
            <template #action>
              <button v-if="item.status == 'open'" @click="acceptEvent(item)" class="btnxs btnPrimary">接受</button>
              <div class="relative">
                <div>
                  <button
                    v-if="item.status == 'accepted' && isCurrentMember(item, memberId)"
                    @click="commitEvent(item)"
                    class="btnxs btnActiveReverse">
                    提交
                  </button>
                  <button
                    v-if="item.status == 'committed' && isCurrentMember(item, memberId) && eventsMatchingByRID"
                    @click="alterCommit(item)"
                    class="btnxs btnWarningReverse">
                    修改
                  </button>
                </div>
              </div>
              <button
                v-if="item.status == 'committed' && store.account.role == 'admin' && checkOnly"
                @click="judgeSubmit(item)"
                class="btnxs btnWarningReverse">
                审核
              </button>
              <div class="uppercase" v-if="item.status == 'cancelled' || item.status == 'closed'">
                {{ item.status }}
              </div>
            </template>
            <template #info>
              <div v-if="isCurrentMember(item, memberId)">
                <div>
                  QQ:<em>{{ item.qq || "无" }}</em>
                </div>
                <div>
                  电话:<em>{{ item.phone || "无" }}</em>
                </div>
              </div>
            </template>
            <template #footer>
              <div class="w-17 truncate">{{ item.model || "无型号" }}</div>
              <span class="text-xs ml-2 textDescription">{{ item.gmtCreate }}</span>
              <button
                v-if="item.status == 'accepted' && isCurrentMember(item, memberId) && eventsMatchingByRID"
                @click="dropEvent(item)"
                class="text-xs font-medium text-warning w-8 p-[1px] rounded ml-2 mb-0.5 border border-warning hover:(bg-warning text-warningContent)">
                放弃
              </button>
            </template>
          </event-card>
          <div v-if="filteredList.length == 0">
            <div class="mb-2 text-center text-gray-400">现在是空的</div>
          </div>
          <div class="py-20"></div>
        </div>
      </scroll-area>
    </div>
    <div class="w-full hidden sm:block">
      <div v-if="eventStore.eventId != undefined">
        <events-detail></events-detail>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue"
import { setEvents, acceptEvent, commitEvent, alterCommit, dropEvent, judgeSubmit } from "./EventActions"
import { TabGroup, TabList, Tab } from "@headlessui/vue"
import ScrollArea from "@/components/ScrollArea/ScrollArea.vue"
import EventCard from "../../components/EventCard/EventCard.vue"
import EventsDetail from "./EventsDetail.vue"
import { isCurrentMember } from "@/utils/event"
import { useAccountStore } from "@/stores/account"
import { useEventStore } from "@/stores/event"
import type { Event } from "@/models/event"
import { searchQuery, roleFilter, filterHandler, filteredList, eventsMatchingByRID } from "./EventActions"

const store = useAccountStore()
const eventStore = useEventStore()
const memberId = ref(store.account.memberId || "")

const reachBottomDistance = 100
let isReachingBottom = false

const onScroll = e => {
  let scrollTop = e.target.scrollTop
  let scrollHeight = e.target.scrollHeight
  let offsetHeight = Math.ceil(e.target.getBoundingClientRect().height)
  let currentHeight = scrollTop + offsetHeight + reachBottomDistance
  if (currentHeight >= scrollHeight && !isReachingBottom) {
    isReachingBottom = true
    console.log("触底")
  }
  if (currentHeight < scrollHeight) {
    isReachingBottom = false
  }
}

const showDetail = (e: Event) => {
  eventStore.eventId = e.eventId
  if (e.member == null) {
    eventStore.mine = false
    return
  }
  eventStore.mine = e.member.memberId == (store.account.memberId || "")
}

setEvents()
</script>

<style></style>
