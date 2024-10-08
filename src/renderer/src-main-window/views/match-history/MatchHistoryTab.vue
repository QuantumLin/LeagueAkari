<template>
  <div class="player-page">
    <PlayerTagEditModal
      :puuid="tab.puuid"
      v-model:show="isShowingTagEditModal"
      @edited="(id) => handleTagEdited(id)"
    />
    <NModal v-model:show="isShowingRankedModal">
      <div class="ranked-modal">
        <RankedDisplay
          v-for="r of tab.rankedStats?.queueMap"
          :key="r.queueType"
          class="ranked"
          :ranked-entry="r"
        />
      </div>
    </NModal>
    <Transition name="fade">
      <div class="player-header-simplified" v-if="shouldShowTinyHeader">
        <div class="header-simplified-inner">
          <LcuImage
            class="small-profile-icon"
            :src="tab.summoner ? profileIconUrl(tab.summoner.profileIconId) : undefined"
          />
          <span class="small-game-name">{{ tab.summoner?.gameName }}</span>
          <span class="small-tag-line">#{{ tab.summoner?.tagLine }}</span>
          <div class="header-simplified-actions">
            <NButton
              round
              class="header-button"
              size="small"
              title="切换到上一页"
              @click="handleLoadPage(tab.matchHistory.page - 1)"
              :disabled="tab.matchHistory.page <= 1 || tab.loading.isLoadingMatchHistory"
              tertiary
            >
              <template #icon>
                <NIcon><NavigateBeforeOutlinedIcon /></NIcon>
              </template>
            </NButton>
            <NButton
              title="切换到下一页"
              size="small"
              round
              class="header-button"
              @click="() => handleLoadPage(tab.matchHistory.page + 1)"
              :disabled="tab.loading.isLoadingMatchHistory"
              tertiary
            >
              <template #icon>
                <NIcon><NavigateNextOutlinedIcon /></NIcon>
              </template>
            </NButton>
            <NButton
              tertiary
              class="header-button"
              title="刷新当前页"
              size="tiny"
              round
              :loading="isSomethingLoading"
              @click="() => handleRefresh()"
            >
              <template #icon>
                <NIcon><RefreshIcon /></NIcon>
              </template>
            </NButton>
          </div>
        </div>
      </div>
    </Transition>
    <NScrollbar x-scrollable ref="scrollRef" @scroll="(e) => handleMainContentScroll(e)">
      <div class="inner-container">
        <div class="profile">
          <div class="header-profile">
            <div class="profile-image">
              <LcuImage
                class="profile-image-icon"
                :src="tab.summoner ? profileIconUrl(tab.summoner.profileIconId) : undefined"
              />
              <div class="profile-image-lv" v-if="tab.summoner">
                {{ tab.summoner.summonerLevel }}
              </div>
            </div>
            <div class="profile-name">
              <CopyableText
                class="game-name"
                :class="{ 'long-name': tab.summoner && tab.summoner.gameName.length >= 12 }"
                :text="summonerName(tab.summoner?.gameName, tab.summoner?.tagLine, '-')"
                >{{ tab.summoner?.gameName || '-' }}</CopyableText
              >
              <span class="tag-line">#{{ tab.summoner?.tagLine || '-' }}</span>
            </div>
          </div>
          <div class="header-ranked" v-if="tab.rankedStats">
            <RankedDisplay
              class="ranked"
              :small="isSmallScreen"
              :ranked-entry="tab.rankedStats?.queueMap['RANKED_SOLO_5x5']"
            />
            <RankedDisplay
              class="ranked"
              :small="isSmallScreen"
              :ranked-entry="tab.rankedStats?.queueMap['RANKED_FLEX_SR']"
            />
            <div class="ranked-more">
              <NButton
                :focusable="false"
                title="更多"
                size="tiny"
                secondary
                @click="isShowingRankedModal = true"
              >
                <template #icon>
                  <MoreHorizFilledIcon />
                </template>
              </NButton>
            </div>
          </div>
          <div class="buttons-container">
            <NButton
              secondary
              class="square-button"
              title="标记"
              @click="() => handleTagPlayer()"
              v-if="!isSelf"
            >
              <template #icon>
                <NIcon><EditIcon /></NIcon>
              </template>
            </NButton>
            <NButton
              secondary
              class="square-button"
              title="刷新"
              :loading="isSomethingLoading"
              @click="() => handleRefresh()"
            >
              <template #icon>
                <NIcon><RefreshIcon /></NIcon>
              </template>
            </NButton>
          </div>
        </div>
        <div class="show-on-smaller-screen">
          <NInputNumber
            size="tiny"
            placeholder=""
            style="width: 48px"
            v-model:value="inputtingPage"
            @blur="handleInputBlur"
            @keyup.enter="() => handleLoadPage(inputtingPage || 1)"
            :disabled="tab.loading.isLoadingMatchHistory"
            :min="1"
            :show-button="false"
          />
          <NButton
            size="tiny"
            title="切换到上一页"
            @click="handleLoadPage(tab.matchHistory.page - 1)"
            :disabled="tab.matchHistory.page <= 1 || tab.loading.isLoadingMatchHistory"
            secondary
            >上一页</NButton
          >
          <NButton
            title="切换到下一页"
            size="tiny"
            @click="() => handleLoadPage(tab.matchHistory.page + 1)"
            :disabled="tab.loading.isLoadingMatchHistory"
            secondary
            >下一页</NButton
          >
          <NSelect
            :value="tab.matchHistory.pageSize"
            @update:value="handleChangePageSize"
            :disabled="tab.loading.isLoadingMatchHistory"
            class="page-select"
            size="tiny"
            style="width: 108px"
            :options="pageSizeOptions"
          ></NSelect>
          <NSelect
            v-if="
              cf.settings.useSgpApi || eds.sgpAvailability.currentSgpServerId !== tab.sgpServerId
            "
            size="tiny"
            :value="tab.matchHistory.queueFilter"
            style="width: 160px"
            @update:value="handleChangeSgpTag"
            :disabled="tab.loading.isLoadingMatchHistory"
            :options="sgpTagOptions"
          ></NSelect>
        </div>
        <div class="content">
          <div class="left">
            <div class="left-content-item">
              <div class="left-content-item-content">
                <div style="display: flex; gap: 4px">
                  <NInputNumber
                    size="tiny"
                    placeholder=""
                    style="flex: 1"
                    v-model:value="inputtingPage"
                    @blur="handleInputBlur"
                    @keyup.enter="() => handleLoadPage(inputtingPage || 1)"
                    :disabled="tab.loading.isLoadingMatchHistory"
                    :min="1"
                    :show-button="false"
                  />
                  <NButton
                    size="tiny"
                    title="切换到上一页"
                    @click="handleLoadPage(tab.matchHistory.page - 1)"
                    :disabled="tab.matchHistory.page <= 1 || tab.loading.isLoadingMatchHistory"
                    secondary
                    >上一页</NButton
                  >
                  <NButton
                    title="切换到下一页"
                    size="tiny"
                    @click="() => handleLoadPage(tab.matchHistory.page + 1)"
                    :disabled="tab.loading.isLoadingMatchHistory"
                    secondary
                    >下一页</NButton
                  >
                  <NSelect
                    :value="tab.matchHistory.pageSize"
                    @update:value="handleChangePageSize"
                    :disabled="tab.loading.isLoadingMatchHistory"
                    class="page-select"
                    size="tiny"
                    style="width: 108px"
                    :options="pageSizeOptions"
                  ></NSelect>
                </div>
                <NSelect
                  v-if="
                    cf.settings.useSgpApi ||
                    eds.sgpAvailability.currentSgpServerId !== tab.sgpServerId
                  "
                  size="tiny"
                  :value="tab.matchHistory.queueFilter"
                  @update:value="handleChangeSgpTag"
                  :disabled="tab.loading.isLoadingMatchHistory"
                  style="margin-top: 8px"
                  :options="sgpTagOptions"
                ></NSelect>
              </div>
            </div>
            <div
              class="left-content-item privacy-private"
              v-if="!isSelf && tab.summoner && tab.summoner.privacy === 'PRIVATE'"
            >
              <div class="left-content-item-title">生涯隐藏</div>
              <div class="left-content-item-content">该玩家设置了生涯不可见</div>
            </div>
            <div class="left-content-item tagged-player" v-if="!isSelf && tab.savedInfo?.tag">
              <div class="left-content-item-title">已被标记的玩家</div>
              <NScrollbar class="tagged-player-n-scrollbar">
                <div class="left-content-item-content">{{ tab.savedInfo.tag }}</div>
              </NScrollbar>
            </div>
            <div class="left-content-item" v-if="analysis.matchHistory">
              <div class="left-content-item-title">总览 (本页)</div>
              <div class="left-content-item-content">
                <div class="stat-item" v-if="app.settings.isInKyokoMode" title="Akari's insight">
                  <span class="stat-item-label">Akari Score</span>
                  <span class="stat-item-content" :class="{ 'n-a': analysis.akariScore === null }">
                    <template v-if="analysis.akariScore !== null">
                      <LeagueAkariSpan bold :text="analysis.akariScore.total.toFixed(2)" />
                    </template>
                    <template v-else>N/A</template>
                  </span>
                </div>
                <div class="stat-item">
                  <span class="stat-item-label">平均 KDA</span>
                  <span class="stat-item-content">{{
                    analysis.matchHistory.summary.averageKda.toFixed(2)
                  }}</span>
                </div>
                <div class="stat-item">
                  <span class="stat-item-label">平均击杀参与率</span>
                  <span class="stat-item-content"
                    >{{
                      (analysis.matchHistory.summary.averageKillParticipationRate * 100).toFixed()
                    }}
                    %</span
                  >
                </div>
                <div class="stat-item">
                  <span class="stat-item-label">平均伤害比</span>
                  <span class="stat-item-content"
                    >{{
                      (
                        analysis.matchHistory.summary.averageDamageDealtToChampionShareOfTeam * 100
                      ).toFixed()
                    }}
                    %</span
                  >
                </div>
                <div class="stat-item">
                  <span class="stat-item-label">平均承受伤害比</span>
                  <span class="stat-item-content"
                    >{{
                      (analysis.matchHistory.summary.averageDamageTakenShareOfTeam * 100).toFixed()
                    }}
                    %</span
                  >
                </div>
                <div class="stat-item">
                  <span class="stat-item-label">平均队伍经济比</span>
                  <span class="stat-item-content"
                    >{{
                      (analysis.matchHistory.summary.averageGoldShareOfTeam * 100).toFixed()
                    }}
                    %</span
                  >
                </div>
                <div class="stat-item">
                  <span class="stat-item-label">平均击杀小兵比</span>
                  <span class="stat-item-content"
                    >{{
                      (analysis.matchHistory.summary.averageCsShareOfTeam * 100).toFixed()
                    }}
                    %</span
                  >
                </div>
                <div class="stat-item">
                  <span class="stat-item-label">胜负</span>
                  <span class="stat-item-content"
                    >{{ analysis.matchHistory.summary.win }} 胜
                    {{ analysis.matchHistory.summary.lose }} 负 ({{
                      (analysis.matchHistory.summary.winRate * 100).toFixed()
                    }}
                    %)
                  </span>
                </div>
                <div class="stat-item" v-if="frequentlyUsedChampions.length">
                  <span class="stat-item-label">英雄使用</span>
                  <div class="stat-item-content-champions">
                    <NPopover v-for="c of frequentlyUsedChampions" :key="c.id">
                      <template #trigger>
                        <div class="champion-slot">
                          <LcuImage
                            style="width: 100%; height: 100%"
                            :src="championIconUrl(c.id)"
                          />
                          <div class="champion-used-count">{{ c.count }}</div>
                        </div>
                      </template>
                      <div class="stat-item-content-champion">
                        <div>{{ gameData.champions[c.id]?.name }} · {{ c.count }} 场</div>
                        <div class="win-lose-box">
                          <span class="win">{{ c.win }} 胜</span>
                          <span class="lose">{{ c.lose }} 负</span>
                          <span>(胜率 {{ (c.winRate * 100).toFixed() }} %)</span>
                        </div>
                      </div>
                    </NPopover>
                  </div>
                </div>
              </div>
            </div>
            <div class="left-content-item" v-if="recentlyTeammates.length">
              <div class="left-content-item-title">近期队友 (本页)</div>
              <div class="left-content-item-content">
                <div
                  class="recently-played-item"
                  v-for="p of recentlyTeammates"
                  :key="p.targetPuuid"
                >
                  <LcuImage
                    style="width: 18px; height: 18px"
                    :src="profileIconUrl(p.targetProfileIconId)"
                  />
                  <div class="name-and-tag" @click="() => handleToSummoner(p.targetPuuid)">
                    <span class="game-name">{{ p.targetGameName }}</span>
                    <span class="tag-line">#{{ p.targetTagLine }}</span>
                  </div>
                  <span class="win-or-lose">{{ p.win }} 胜 {{ p.lose }} 负</span>
                </div>
              </div>
            </div>
          </div>
          <div class="right" ref="rightEl">
            <MatchHistoryCard
              class="match-history-card-item"
              @set-show-detailed-game="handleToggleShowDetailedGame"
              @load-detailed-game="(gameId) => mhm.fetchTabDetailedGame(tab.puuid, gameId)"
              @to-summoner="(puuid) => handleToSummoner(puuid)"
              :self-puuid="tab.puuid"
              :is-detailed="g.isDetailed"
              :is-loading="g.isLoading"
              :is-expanded="g.isExpanded"
              :game="g.game"
              v-for="g of tab.matchHistory.games"
              :key="g.game.gameId"
            />
            <div class="match-history-empty-placeholder" v-if="!tab.matchHistory.games.length">
              <NSpin v-if="tab.loading.isLoadingMatchHistory" />
              <span v-else>暂无数据</span>
            </div>
          </div>
        </div>
      </div>
    </NScrollbar>
  </div>
</template>

<script setup lang="ts">
import CopyableText from '@shared/renderer/components/CopyableText.vue'
import LcuImage from '@shared/renderer/components/LcuImage.vue'
import LeagueAkariSpan from '@shared/renderer/components/LeagueAkariSpan.vue'
import { useAppStore } from '@shared/renderer/modules/app/store'
import { useCoreFunctionalityStore } from '@shared/renderer/modules/core-functionality/store'
import { useExternalDataSourceStore } from '@shared/renderer/modules/external-data-source/store'
import { championIconUrl, profileIconUrl } from '@shared/renderer/modules/game-data'
import { useGameDataStore } from '@shared/renderer/modules/lcu-state-sync/game-data'
import { laNotification } from '@shared/renderer/notification'
import {
  analyzeMatchHistory,
  analyzeMatchHistoryPlayers,
  calculateAkariScore
} from '@shared/utils/analysis'
import { summonerName } from '@shared/utils/name'
import { Edit20Filled as EditIcon } from '@vicons/fluent'
import { RefreshSharp as RefreshIcon } from '@vicons/ionicons5'
import {
  MoreHorizFilled as MoreHorizFilledIcon,
  NavigateBeforeOutlined as NavigateBeforeOutlinedIcon,
  NavigateNextOutlined as NavigateNextOutlinedIcon
} from '@vicons/material'
import { useMediaQuery } from '@vueuse/core'
import {
  NButton,
  NIcon,
  NInputNumber,
  NModal,
  NPopover,
  NScrollbar,
  NSelect,
  NSpin
} from 'naive-ui'
import { computed, nextTick, ref, watch } from 'vue'

import PlayerTagEditModal from '@main-window/components/PlayerTagEditModal.vue'
import { matchHistoryTabsRendererModule as mhm } from '@main-window/modules/match-history-tabs'
import { TabState, useMatchHistoryTabsStore } from '@main-window/modules/match-history-tabs/store'

import MatchHistoryCard from './card/MatchHistoryCard.vue'
import RankedDisplay from './widgets/RankedDisplay.vue'

const props = withDefaults(
  defineProps<{
    tab: TabState & { id: string }
    isSelf?: boolean
  }>(),
  {
    isSelf: false
  }
)

// 1182px - is same in which defined in CSS
const isSmallScreen = useMediaQuery(`(max-width: 1182px)`)

const analysis = computed(() => {
  const matchHistory = analyzeMatchHistory(props.tab.matchHistory.games, props.tab.puuid)
  const players = analyzeMatchHistoryPlayers(props.tab.matchHistory.games, props.tab.puuid)

  return {
    matchHistory: matchHistory,
    playerRelationship: players,
    akariScore: matchHistory ? calculateAkariScore(matchHistory) : null
  }
})

const eds = useExternalDataSourceStore()

const scrollRef = ref()
const rightEl = ref()

const mh = useMatchHistoryTabsStore()
const cf = useCoreFunctionalityStore()
const gameData = useGameDataStore()
const app = useAppStore()

const handleToggleShowDetailedGame = (gameId: number, expand: boolean) => {
  mh.setMatchHistoryExpand(props.tab.id, gameId, expand)
}

const isShowingRankedModal = ref(false)

const isSomethingLoading = computed(() => {
  return (
    props.tab.loading.isLoadingMatchHistory ||
    props.tab.loading.isLoadingRankedStats ||
    props.tab.loading.isLoadingSummoner
  )
})

const scrollToRightElTop = () => {
  const top = rightEl.value?.offsetTop
  if (top < mainContentScrollTop.value) {
    scrollRef.value?.scrollTo({ top: top })
  }
}

const handleRefresh = async () => {
  try {
    await mhm.fetchTabFullData(props.tab.id)
    scrollToRightElTop()
  } catch {
    laNotification.warn('召唤师信息', `无法拉取用户 ${props.tab.id} 的信息`)
  }
}

const handleLoadPage = async (page?: number) => {
  const r = await mhm.fetchTabMatchHistory(props.tab.id, page)
  scrollToRightElTop()
  return r
}

const inputtingPage = ref(props.tab.matchHistory.page)
const handleInputBlur = () => {
  inputtingPage.value = props.tab.matchHistory.page
}

const pageSizeOptions = [
  {
    label: '10 项',
    value: 10
  },
  {
    label: '20 项',
    value: 20
  },
  {
    label: '50 项',
    value: 50
  },
  {
    label: '100 项',
    value: 100
  }
]

const handleChangePageSize = async (pageSize: number) => {
  const r = await mhm.fetchTabMatchHistory(
    props.tab.id,
    props.tab.matchHistory.page,
    pageSize,
    props.tab.matchHistory.queueFilter
  )
  return r
}

watch(
  () => props.tab.matchHistory.page,
  (page) => {
    inputtingPage.value = page
  }
)

const sgpTagOptions = computed(() => {
  return [
    {
      label: '所有队列',
      value: -1
    },
    {
      label: gameData.queues[420]?.name || 'Ranked Solo/Duo',
      value: 420
    },
    {
      label: gameData.queues[430]?.name || 'Normal',
      value: 430
    },
    {
      label: gameData.queues[440]?.name || 'Ranked Flex',
      value: 440
    },
    {
      label: gameData.queues[450]?.name || 'ARAM',
      value: 450
    },

    {
      label: gameData.queues[1700]?.name || 'ARENA',
      value: 1700
    },
    {
      label: gameData.queues[490]?.name || 'Quickplay',
      value: 490
    },
    {
      label: gameData.queues[1900]?.name || 'URF',
      value: 1900
    },
    {
      label: gameData.queues[900]?.name || 'ARURF',
      value: 900
    }
  ]
})

const handleChangeSgpTag = async (queueFilter: number | string) => {
  const r = await mhm.fetchTabMatchHistory(
    props.tab.id,
    1,
    props.tab.matchHistory.pageSize,
    queueFilter
  )
  return r
}

const isShowingTagEditModal = ref(false)
const handleTagPlayer = async () => {
  isShowingTagEditModal.value = true
}
const handleTagEdited = (_puuid: string) => {
  mhm.querySavedInfo(props.tab.id)
}

const FREQUENT_USE_CHAMPION_THRESHOLD = 1
const RECENTLY_PLAYED_PLAYER_THRESHOLD = 2

const frequentlyUsedChampions = computed(() => {
  const a = analysis.value.matchHistory
  if (!a) {
    return []
  }

  return Object.values(a.champions)
    .filter((c) => c.count >= FREQUENT_USE_CHAMPION_THRESHOLD)
    .sort((a, b) => {
      if (a.count !== b.count) {
        return b.count - a.count
      }

      return b.win - a.win
    })
})

const recentlyTeammates = computed(() => {
  const relationship = analysis.value.playerRelationship
  const teammates = Object.values(relationship)
    .filter((a) => a.games.length >= RECENTLY_PLAYED_PLAYER_THRESHOLD)
    .map((a) => {
      const teammateGames = a.games.filter((g) => !g.isOpponent)
      return {
        ...a,
        games: teammateGames
      }
    })
    .filter((a) => a.games.length >= RECENTLY_PLAYED_PLAYER_THRESHOLD)
    .map((a) => {
      const win = a.games.filter((g) => g.win).length
      const lose = a.games.filter((g) => !g.win).length
      return { ...a, win, lose }
    })
    .sort((a, b) => {
      if (a.games.length !== b.games.length) {
        return b.games.length - a.games.length
      }

      return b.win - a.win
    })

  return teammates
})

const { navigateToTab } = mhm.useNavigateToTab()

const handleToSummoner = (puuid: string) => {
  const { sgpServerId } = mhm.parseUnionId(props.tab.id)
  navigateToTab(puuid, sgpServerId)
}

const SHOW_TINY_HEADER_THRESHOLD = 160
const mainContentScrollTop = ref(0)
const handleMainContentScroll = (e: Event) => {
  mainContentScrollTop.value = (e.target as HTMLElement).scrollTop
}

const shouldShowTinyHeader = computed(() => mainContentScrollTop.value > SHOW_TINY_HEADER_THRESHOLD)

// workaround: KeepAlive 下 Naive UI 滚动条复位问题
watch(
  () => mh.currentTab?.id,
  () => {
    if (mh.currentTab?.id === props.tab.puuid) {
      nextTick(() => {
        scrollRef.value?.scrollTo({ top: mainContentScrollTop.value })
      })
    }
  }
)

defineExpose({
  puuid: props.tab.puuid,
  refresh: handleRefresh
})
</script>

<style lang="less" scoped>
@container-width: 1064px;

.player-page {
  position: relative;
  height: 100%;
}

.ranked-modal {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  border-radius: 4px;
  background-color: #202020;
  padding: 16px;

  .ranked {
    background-color: #f4f4f40e;
  }
}

.profile {
  display: flex;
  align-items: center;
  height: 140px;
  box-sizing: border-box;
  padding: 20px 20px 12px 20px;
}

.player-header-simplified {
  position: absolute;
  left: 0;
  right: 0;
  height: 64px;
  z-index: 1;
  background-color: rgba(#202020, 0.95);
  box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(4px);

  .header-simplified-inner {
    display: flex;
    align-items: center;
    height: 100%;
    width: @container-width;
    padding: 0 16px;
    box-sizing: border-box;
    margin: 0 auto;

    .small-profile-icon {
      width: 32px;
      height: 32px;
      border-radius: 4px;
    }

    .small-game-name {
      font-size: 16px;
      margin-left: 8px;
    }

    .small-tag-line {
      position: relative;
      top: 2px;
      font-size: 14px;
      margin-left: 6px;
      color: #858585;
    }

    @media (max-width: 1182px) {
      width: 764px;
    }
  }

  .header-simplified-actions {
    display: flex;
    gap: 8px;
    margin-left: auto;
  }
}

.inner-container {
  height: 100%;
  width: @container-width;
  margin: 0 auto;

  .content {
    display: flex;
  }

  .show-on-smaller-screen {
    display: none;
    padding: 0px 12px;

    @media (max-width: 1182px) {
      display: flex;
      justify-content: flex-end;
      gap: 4px;
    }
  }

  @media (max-width: 1182px) {
    width: 764px;
  }
}

.content .left {
  position: relative;
  flex: 1;
  padding: 12px 0 12px 12px;

  @media (max-width: 1182px) {
    display: none;
  }
}

.left-content-item {
  padding: 8px 16px;
  margin-bottom: 8px;
  background-color: #ffffff10;
  border-radius: 4px;

  .left-content-item-title {
    font-size: 16px;
    font-weight: bold;
    margin-bottom: 4px;
  }

  .left-content-item-content {
    font-size: 13px;
    color: #dcdcdc;
  }
}

.left-content-item.privacy-private {
  background-color: #781f1f60;
}

.left-content-item.tagged-player {
  background-color: #00407d60;

  .left-content-item-content {
    white-space: pre-wrap;
  }

  :deep(.tagged-player-n-scrollbar) {
    max-height: 100px;
  }
}

.content .right {
  padding: 12px 12px;

  .right-content-title {
    display: flex;
    align-items: center;
    height: 40px;
    font-size: 22px;
    font-weight: bold;
    margin-bottom: 16px;
    color: #dcdcdc;
  }

  .match-history-empty-placeholder {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 200px;
    width: 740px;
    background-color: #ffffff05;
    color: rgb(83, 83, 83);
  }

  .match-history-card-item {
    margin-bottom: 4px;
  }
}

.header-profile {
  display: flex;
  flex: 1;
  height: 64px;

  .profile-image {
    position: relative;
    width: 64px;
    height: 64px;
  }

  .profile-image .profile-image-icon {
    width: 100%;
    height: 100%;
    border-radius: 4px;
    margin-bottom: 2px;
  }

  .profile-image .profile-image-lv {
    font-size: 10px;
    color: #fff;
    position: absolute;
    bottom: -4px;
    right: -4px;
    background-color: #00000060;
    padding: 2px 4px;
    border-radius: 4px;
  }

  .profile-name {
    display: flex;
    flex-direction: column;
    align-self: center;
    margin-left: 12px;
  }

  .profile-name .game-name {
    font-size: 20px;
    color: #fff;
    font-weight: bold;
  }

  .profile-name .game-name.long-name {
    font-size: 14px;
  }

  .profile-name .tag-line {
    position: relative;
    font-size: 14px;
    color: #858585;
  }
}

.header-ranked {
  position: relative;
  display: flex;
  gap: 12px;

  .ranked-more {
    position: absolute;
    bottom: -6px;
    right: -8px;
  }

  .ranked {
    background-color: #ffffff04;
  }
}

.stat-item {
  display: flex;
  width: 100%;
  align-items: center;
  gap: 8px;

  &:not(:last-child) {
    margin-bottom: 2px;
  }

  .stat-item-label {
    font-size: 12px;
    color: #a2a2a2;
  }

  .stat-item-content {
    margin-left: auto;
    font-size: 13px;
    color: #fff;
    text-align: right;

    &.n-a {
      filter: brightness(0.6);
    }
  }

  .stat-item-content-champions {
    max-width: 110px;
    margin-left: auto;
    display: flex;
    flex-wrap: wrap;
    gap: 2px;
    justify-content: end;

    .champion-slot {
      position: relative;
      width: 20px;
      height: 20px;
    }

    .champion-used-count {
      position: absolute;
      bottom: -4px;
      right: -2px;
      font-size: 10px;
      color: #d3d3d3;
      background-color: #00000060;
      padding: 0 2px;
      border-radius: 2px;
    }
  }
}

.recently-played-item {
  display: flex;
  align-items: center;

  &:not(:last-child) {
    margin-bottom: 4px;
  }

  .name-and-tag {
    display: flex;
    cursor: pointer;
  }

  .name-and-tag:hover {
    .game-name,
    .tag-line {
      filter: brightness(1.2);
    }
  }

  .game-name {
    font-size: 12px;
    color: #fff;
    margin-left: 4px;
    max-width: 120px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    transition: all 0.2s;
  }

  .tag-line {
    font-size: 11px;
    color: #858585;
    margin-left: 2px;
    transition: all 0.2s;
  }

  .win-or-lose {
    margin-left: auto;
    font-size: 12px;
    color: #acacac;
  }
}

.buttons-container {
  display: flex;
  margin-left: 32px;
  gap: 8px;
  justify-content: flex-end;
}

.square-button {
  width: 42px;
  height: 42px;
}

.header-button {
  width: 32px;
  height: 32px;
}

.stat-item-content-champion {
  font-size: 12px;

  .win-lose-box {
    display: flex;
    gap: 4px;
    margin-top: 2px;
  }

  .win {
    color: #239b23;
  }

  .lose {
    color: #c76713;
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.fade-enter-to,
.fade-leave-from {
  opacity: 1;
}
</style>
