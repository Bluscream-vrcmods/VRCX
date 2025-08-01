<template>
    <div class="x-friend-item" @click="$emit('click')">
        <template v-if="friend.ref">
            <div
                class="avatar"
                :class="isFriendActiveOrOffline ? undefined : userStatusClass(friend.ref, friend.pendingOffline)">
                <img v-lazy="userImage(friend.ref, true)" />
            </div>
            <div class="detail">
                <span v-if="!hideNicknames && friend.$nickName" class="name" :style="{ color: friend.ref.$userColour }">
                    {{ friend.ref.displayName }} ({{ friend.$nickName }})
                </span>
                <span v-else class="name" :style="{ color: friend.ref.$userColour }"
                    >{{ friend.ref.displayName }}{{ isGroupByInstance && friend.isVIP ? ' ⭐' : '' }}</span
                >

                <span v-if="isFriendActiveOrOffline" class="extra">{{ friend.ref.statusDescription }}</span>
                <template v-else>
                    <span v-if="friend.pendingOffline" class="extra">
                        <i class="el-icon-warning-outline" /> {{ $t('side_panel.pending_offline') }}
                    </span>
                    <template v-else-if="isGroupByInstance">
                        <i v-if="isFriendTraveling" class="el-icon el-icon-loading"></i>
                        <Timer
                            class="extra"
                            :epoch="epoch"
                            :style="isFriendTraveling ? { display: 'inline-block', overflow: 'unset' } : undefined" />
                    </template>
                    <Location
                        v-else
                        class="extra"
                        :location="friend.ref.location"
                        :traveling="friend.ref.travelingToLocation"
                        :link="false" />
                </template>
            </div>
        </template>
        <template v-else-if="!friend.ref && !isRefreshFriendsLoading">
            <span>{{ friend.name || friend.id }}</span>
            <el-button
                ttype="text"
                icon="el-icon-close"
                size="mini"
                style="margin-left: 5px"
                @click.stop="$emit('confirm-delete-friend', friend.id)">
            </el-button>
        </template>

        <el-skeleton v-else animated class="skeleton" :throttle="100">
            <template slot="template">
                <div>
                    <el-skeleton-item variant="circle" />
                    <div>
                        <el-skeleton-item variant="text" />
                        <el-skeleton-item variant="text" />
                    </div>
                </div>
            </template>
        </el-skeleton>
    </div>
</template>

<script setup>
    import { storeToRefs } from 'pinia';
    import { computed } from 'vue';
    import { userImage, userStatusClass } from '../shared/utils';
    import { useAppearanceSettingsStore, useFriendStore } from '../stores';

    const props = defineProps({
        friend: { type: Object, required: true },
        isGroupByInstance: Boolean
    });

    const { hideNicknames } = storeToRefs(useAppearanceSettingsStore());
    const { isRefreshFriendsLoading } = storeToRefs(useFriendStore());

    const isFriendTraveling = computed(() => props.friend.ref?.location === 'traveling');
    const isFriendActiveOrOffline = computed(() => props.friend.state === 'active' || props.friend.state === 'offline');
    const epoch = computed(() =>
        isFriendTraveling.value ? props.friend.ref?.$travelingToTime : props.friend.ref?.$location_at
    );
</script>

<style scoped>
    .skeleton {
        height: 40px;
        width: 100%;
        ::v-deep .el-skeleton {
            height: 100%;
            > div {
                height: 100%;
                display: flex;
                align-items: center;
                > :first-child {
                    margin-right: 8px;
                    height: 40px;
                    width: 40px;
                }
                > :last-child {
                    flex: 1;
                    display: flex;
                    flex-direction: column;
                    justify-content: center;
                    > :first-child {
                        width: 50%;
                        margin-bottom: 4px;
                    }
                    > :last-child {
                        width: 90%;
                    }
                }
            }
        }
    }
</style>
