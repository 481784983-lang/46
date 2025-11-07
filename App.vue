<template>
  <div class="new-chinese-ui-container">
    <!-- 标签导航 -->
    <div class="tab-navigation">
      <button
        v-for="tab in tabs"
        :key="tab.key"
        class="tab-btn"
        :class="{ active: activeTab === tab.key }"
        @click="activeTab = tab.key"
      >
        {{ tab.name }}
      </button>
    </div>

    <!-- 标签内容 -->
    <div class="tab-content">
      <!-- 世界信息标签 -->
      <div v-if="activeTab === 'world'" class="tab-panel">
        <WorldInfo :world-data="gameData.世界" />
      </div>

      <!-- 主角信息标签 -->
      <div v-if="activeTab === 'character'" class="tab-panel">
        <CharacterPanel :character-data="gameData.主角" @update-attribute="updateAttribute" />
      </div>

      <!-- 行囊标签 -->
      <div v-if="activeTab === 'inventory'" class="tab-panel">
        <InventoryPanel :inventory-data="gameData.行囊" />
      </div>

      <!-- 关系标签 -->
      <div v-if="activeTab === 'relationships'" class="tab-panel">
        <RelationshipsPanel :relationships="gameData.关系表" :allies="gameData.结缘录" />
      </div>

      <!-- 天下闻标签 -->
      <div v-if="activeTab === 'tianxia'" class="tab-panel">
        <TianxiaPanel :tianxia-data="gameData.天下闻" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, onUnmounted } from 'vue';
import WorldInfo from './components/WorldInfo.vue';
import CharacterPanel from './components/CharacterPanel.vue';
import InventoryPanel from './components/InventoryPanel.vue';
import RelationshipsPanel from './components/RelationshipsPanel.vue';
import TianxiaPanel from './components/TianxiaPanel.vue';

const activeTab = ref('character'); // 默认显示主角信息

// 标签配置
const tabs = [
  { key: 'world', name: '世界' },
  { key: 'character', name: '主角' },
  { key: 'inventory', name: '行囊' },
  { key: 'relationships', name: '关系' },
  { key: 'tianxia', name: '天下闻' }
];

const gameData = reactive({
  "世界": {
    "时间": ["", "当前的游戏内日期。随剧情推进更新，格式为：XXN年 (公元 YYYY年)。请AI根据剧情（如战斗、旅行）合理推进时间。"],
    "地点": ["", "主角<user>当前所在的具体地点。e.g., 洛阳->虎牢关"]
  },
  "主角": {
    "势力": ["群雄", "【UI主题核心】主角<user>当前所属势力。UI将根据此值自动切换主题 ('群雄', '炎汉', '魏', '吴')。"],
    "声望": ["无名之辈", "主角<user>在天下的声望等级。随重大事件更新。e.g., 无名之辈 -> 崭露头角 -> 威震一方 -> 天下闻名 -> 人道魁首"],
    "命": [1000, "【最大生命值】。AI应根据'武力'、'血脉'等进行设定。"],
    "法": [1000, "【最大法力值】。AI应根据'谋略'、'血脉'等进行设定。"],
    "武力": [0, "[1-100]，主角<user>的武力值。随锻炼、奇遇或血脉觉醒更新。"],
    "谋略": [0, "[1-100]，主角<user>的谋略值。随学习或经历更新。"],
    "魅力": [0, "[1-100]，主角<user>的魅力值。随事迹或血脉觉醒更新。"],
    "官职": ["无", "主角的官职，由汉室(曹婥)册封，影响声望和权限。"],
    "血脉": ["凡人", "主角觉醒的血脉。e.g., 凡人, 亚龙血脉, 魔裔。"],
    "仙术": [
      {
        "$meta": {
          "extensible": true,
          "template": {
            "名称": ["新仙术", "【类型】"],
            "类型": ["未知", "e.g., 奇门, 符咒, 召唤"],
            "消耗": ["法力 0", "发动消耗"],
            "描述": ["暂无描述。", ""]
          }
        }
      },
      "主角学会的仙术/奇门。"
    ]
  },
  "行囊": {
    "装备": {
      "武器": ["无", "当前装备的武器。"],
      "防具": ["无", "当前装备的防具。"],
      "饰品": ["无", "当前装备的饰品。"]
    },
    "背包": [
      {
        "$meta": {
          "extensible": true,
          "template": {
            "名称": ["新物品", "【消耗品】"],
            "数量": [1, "描述：..."],
            "描述": ["暂无描述。", ""]
          }
        }
      },
      "存放消耗品、材料、任务道具。"
    ]
  },
  "关系表": [
    { "$meta": { "extensible": true } },
    "一个对象，用于【快速查阅】<user>已建立的【特殊羁绊】。键是人名(snake_case)，值是关系。AI应在'结缘录.xxx.当前关系'更新为'至交'、'妻子'、'宿敌'等特殊关系时，【同步】更新此表。"
  ],
  "结缘录": [
    {
      "$meta": {
        "extensible": true,
        "required": [
          "姓名",
          "年龄",
          "命",
          "法",
          "好感度",
          "仙术",
          "背包"
        ],
        "template": {
          "姓名": ["？？？", "字？？？"],
          "年龄": [0, "？？？"],
          "官职": ["无", "？？？"],
          "武力": [0, "[1-100+]"],
          "谋略": [0, "[1-100+]"],
          "魅力": [0, "[1-100+]"],
          "命": [100, "【最大生命值】"],
          "法": [100, "【最大法力值】"],
          "坐骑": ["无", "？？？"],
          "衣着": ["暂无描述。", "当前装束"],
          "外貌": ["暂无描述。", "外貌特征"],
          "武器": ["【凡品】无", "？？？"],
          "好感度": [0, "[-1000 - +1000] (陌生)"],
          "当前关系": ["陌生", "？？？"],
          "仙术": [
            {
              "$meta": {
                "extensible": true,
                "template": {
                  "名称": ["新仙术", "【血脉】"],
                  "消耗": ["法力 0", ""],
                  "描述": ["暂无描述。", ""]
                }
              }
            },
            "‘将姫’的血脉能力与武技。"
          ],
          "背包": [
            {
              "$meta": {
                "extensible": true,
                "template": {
                  "名称": ["新物品", "【消耗品】"],
                  "数量": [1, "描述：..."],
                  "描述": ["暂无描述。", ""]
                }
              }
            },
            "‘将姫’的随身行囊。"
          ]
        }
      }
    },
    "此表记录所有重要NPC的【详细数据】。当<user>遇到新'将姫'时，AI应按此格式添加新条目。"
  ],
  "天下闻": {
    "奇闻": ["（暂无奇闻）", "【V4.0规则】战略总览 (已确认的情报)。用 \\n 换行，记录【将姫】的军队动向和【神魔】的踪迹。e.g., \n【将姫】曹婥(女)已于陈留起兵。\n【神魔】'魔王'董妵(女)已抵达洛阳。"],
    "当前剧情": ["（等待开局...）", "【V4.0规则】<user>的实时冒险日志。总结<user>刚做了什么，和下一步的目标。"],
    "咏叹": ["（暂无诗篇）", "AI在剧情关键节点创作的七言律诗，总结宿命与史诗。"]
  }
});

function updateAttribute(attribute: string, change: number) {
  const currentValue = ((gameData.主角 as any)[attribute] as [number, string])[0];
  const newValue = Math.max(0, Math.min(100, currentValue + change));
  ((gameData.主角 as any)[attribute] as [number, string])[0] = newValue;
  debouncedSave();
}

let saveTimer: number | null = null;

// 加载游戏数据
function loadGameData() {
  try {
    const messageId = getCurrentMessageId();
    if (messageId) {
      const variables = getVariables({ type: 'message', message_id: messageId });
      if (variables && variables.stat_data) {
        // 更新主角数据
        if (variables.stat_data.主角) {
          Object.assign(gameData.主角, variables.stat_data.主角);
        }
        // 更新行囊数据
        if (variables.stat_data.行囊) {
          Object.assign(gameData.行囊, variables.stat_data.行囊);
        }
        // 更新关系数据
        if (variables.stat_data.关系表) {
          gameData.关系表 = { ...variables.stat_data.关系表 };
        }
        if (variables.stat_data.结缘录) {
          gameData.结缘录 = [...variables.stat_data.结缘录];
        }
        // 更新天下闻数据
        if (variables.stat_data.天下闻) {
          Object.assign(gameData.天下闻, variables.stat_data.天下闻);
        }
        console.log('游戏数据已更新');
      }
    }
  } catch (error) {
    console.error('加载游戏数据时出错:', error);
  }
}

// 保存游戏数据
function saveGameData() {
  try {
    const messageId = getCurrentMessageId();
    if (messageId) {
      replaceVariables(gameData, { type: 'message', message_id: messageId });
    }
  } catch (error) {
    console.error('保存游戏数据时出错:', error);
  }
}

// 防抖保存
function debouncedSave() {
  if (saveTimer) {
    clearTimeout(saveTimer);
  }
  saveTimer = window.setTimeout(saveGameData, 1000);
}

onMounted(() => {
  loadGameData();

  // 每5秒自动刷新一次数据
  const intervalId = setInterval(() => {
    if (getCurrentMessageId()) {
      loadGameData();
    }
  }, 5000);

  onUnmounted(() => {
    clearInterval(intervalId);
    if (saveTimer) {
      clearTimeout(saveTimer);
    }
  });
});
</script>

<style lang="scss" scoped>
.new-chinese-ui-container {
  width: 100%;
  min-height: 600px;
  background: linear-gradient(135deg, var(--background-color) 0%, #e8dcc0 100%);
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.15);
  font-family: 'Microsoft YaHei', 'SimSun', serif;
  position: relative;
  overflow: hidden;

  // 古代纹理装饰
  &::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-image: url('data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><defs><pattern id="chinese-pattern" x="0" y="0" width="20" height="20" patternUnits="userSpaceOnUse"><circle cx="10" cy="10" r="1" fill="rgba(139,69,19,0.1)"/></pattern></defs><rect width="100" height="100" fill="url(%23chinese-pattern)"/></svg>');
    background-size: 30px 30px;
    opacity: 0.3;
    pointer-events: none;
    z-index: -1;
  }

  // 中式边框装饰
  &::after {
    content: '';
    position: absolute;
    top: 10px;
    left: 10px;
    right: 10px;
    bottom: 10px;
    border: 2px solid rgba(139, 69, 19, 0.2);
    border-radius: 12px;
    pointer-events: none;
    z-index: -1;
  }
}

.main-layout {
  display: grid;
  grid-template-columns: 320px 1fr 320px;
  gap: 24px;
  position: relative;
  z-index: 1;

  .left-panel,
  .center-panel,
  .right-panel {
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  .center-panel {
    gap: 20px;
  }
}

// 标签导航样式
.tab-navigation {
  display: flex;
  justify-content: center;
  margin-bottom: 24px;
  border-bottom: 2px solid var(--border-color);
  padding-bottom: 12px;
}

.tab-btn {
  background: rgba(255, 255, 255, 0.8);
  border: 2px solid var(--border-color);
  border-radius: 8px 8px 0 0;
  padding: 12px 24px;
  margin: 0 4px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 16px;
  font-weight: bold;
  color: var(--text-color);

  &:hover {
    background: rgba(139, 69, 19, 0.1);
    transform: translateY(-2px);
  }

  &.active {
    background: var(--primary-color);
    color: white;
    border-color: var(--primary-color);
    box-shadow: 0 4px 12px rgba(139, 69, 19, 0.3);
  }
}

// 标签内容样式
.tab-content {
  width: 100%;
}

.tab-panel {
  display: flex;
  justify-content: center;
  min-height: 400px;
}

// 响应式设计
@media (max-width: 1200px) {
  .tab-navigation {
    flex-wrap: wrap;
  }

  .tab-btn {
    padding: 10px 16px;
    font-size: 14px;
  }
}

@media (max-width: 768px) {
  .new-chinese-ui-container {
    padding: 16px;
  }

  .tab-navigation {
    margin-bottom: 16px;
  }

  .tab-btn {
    padding: 8px 12px;
    font-size: 14px;
    margin: 0 2px;
  }

  .tab-panel {
    min-height: 300px;
  }
}
</style>
