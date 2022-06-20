<template>
    <audio id="GameBGM" src="BGM/ElementCard.mp3" loop></audio>
    <audio id="GameSE"></audio>
    <div class="dialog" :class="dialogClass">
        <div class="dialogUI">
            <p>{{dialogMsg}}</p>
            <button onclick="location.reload()">重新开始</button>
        </div>
    </div>
    <div class="Game">
        <div class="GameUI">
            <div class="leftUI">
                <div class="leftUI_background">
                    <div class="exit"><p>退出</p></div>
                    <div class="resetcard" :class="resetCardButtonClass" @click="ResetCard(-3)">
                        <div class="resetcard_background"></div>
                        <div class="resetcardUI">
                            <div class="resetcardUI_left">
                                <div class="cardback">
                                    <div class="cardback_pattern">
                                        <p>{{remainCardCount}}</p>
                                    </div>
                                </div>
                            </div>
                            <div class="resetcardUI_right">
                                <div class="resetcardUI_righttop">
                                    <p>重新发牌</p>
                                </div>
                                <div class="resetcardUI_rightdown">
                                    <span></span><p>-3</p>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="skilllist" id="skilllist">
                        <p>--> 技能 &lt--</p>
                        <div class="skill" :class="RetrunSkillClass(skill.skill_1_cd)" @click="skill.skill_1">
                            <div class="skill_background"></div>
                            <div class="skill_UI">
                                <div class="skillimg skill_1"><p v-if="skill.skill_1_cd != 0">{{skill.skill_1_cd}}</p></div>
                                <div class="skillinfo"><p style="color: rgb(56,50,76);">旋风剑</p><p v-if="skill.skill_1_cd != 0" style="color: rgb(120,87,66);">{{skill.skill_1_cd}}回合后可用</p></div>
                            </div>
                        </div>
                        <div class="skill" :class="RetrunSkillClass(skill.skill_2_cd)" @click="skill.skill_2">
                            <div class="skill_background"></div>
                            <div class="skill_UI">
                                <div class="skillimg skill_2"><p v-if="skill.skill_2_cd != 0">{{skill.skill_2_cd}}</p></div>
                                <div class="skillinfo"><p style="color: rgb(56,50,76);">行动力恢复</p><p v-if="skill.skill_2_cd != 0" style="color: rgb(120,87,66);">{{skill.skill_2_cd}}回合后可用</p></div>
                            </div>
                        </div>
                        <div class="skill" :class="RetrunSkillClass(skill.skill_3_cd)" @click="skill.skill_3">
                            <div class="skill_background"></div>
                            <div class="skill_UI">
                                <div class="skillimg skill_3"><p v-if="skill.skill_3_cd != 0">{{skill.skill_3_cd}}</p></div>
                                <div class="skillinfo"><p style="color: rgb(56,50,76);">生命值恢复</p><p v-if="skill.skill_3_cd != 0" style="color: rgb(120,87,66);">{{skill.skill_3_cd}}回合后可用</p></div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="rightUI">
                <div class="statusbar">
                    <div class="gamescore">
                        <span></span><p style="text-shadow:3px 3px rgb(92,90,163); color: whitesmoke;">{{gameScore}}</p>
                    </div>
                    <div class="actionpoint">
                        <span></span><p style="text-shadow:3px 3px rgb(92,90,163); color: whitesmoke;">{{actionPoint}}</p><p> / {{maxActionPoint}}</p>
                    </div>
                    <div class="healthpoint">
                        <span></span><p style="text-shadow:3px 3px rgb(92,90,163); color: whitesmoke;">{{healthPoint}}</p><p> / {{maxHealthPoint}}</p>
                    </div>
                    <div class="buttongroup">
                        <div class="sebutton" @click="SwitchSEState">
                            <p>音效: {{seState}}</p>
                        </div>
                        <div class="bgmbutton" @click="SwitchBGMState">
                            <p>音乐: {{bgmState}}</p>
                        </div>
                    </div>
                </div>
                <div class="carddesktop">
                    <div class="desktop_background" id="desktop_background">
                        <div class="card" :class="ReturnCardClassName(cardName)" v-for="(cardName, index) in desktopCardList_Change" @click.self="ClickCard($event, index)">
                              <div class="card_background" v-if="ReturnCardBool(cardName)">
                                  <div class="cardimg" :class="ReturnImgClassName(cardName)"></div>
                                  <div class="cardpoint">{{ReturnPoint(cardName)}}</div>
                                  <div class="cardtext">{{ReturnText(cardName)}}</div>
                              </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
  name: 'App',
  watch: {
    bgmState(val) {
      if(val == '开')
      {
        this.sound_BGM.play();
      }
      else
      {
        this.sound_BGM.pause();
      }
    },
    gameState(val) {
      if(!val)
      {
        this.bgmState = '关';
        if(this.healthPoint <= 0)
        {
          this.PlaySE(this.sound_Url.lost);
          this.dialogMsg = '游戏失败';
          this.dialogClass = 'dialog_show';
        }
        else
        {
          this.PlaySE(this.sound_Url.win);
          this.dialogMsg = '游戏胜利';
          this.dialogClass = 'dialog_show';
        }
      }
    },
    'skill.skill_1_cd'(val) {
      if(val < 0)
      {
        this.skill.skill_1_cd = 0;
      }
    },
    'skill.skill_2_cd'(val) {
      if(val < 0)
      {
        this.skill.skill_2_cd = 0;
      }
    },
    'skill.skill_3_cd'(val) {
      if(val < 0)
      {
        this.skill.skill_3_cd = 0;
      }
    },
    healthPoint(val) {
      if(val > this.maxHealthPoint)
      {
        this.healthPoint = this.maxHealthPoint;
      }
    },
    actionPoint(val) {
      if(val > this.maxActionPoint)
      {
        this.actionPoint = this.maxActionPoint;
      }
    },
    remainCardCount(val) {
      if(val == 0)
      {
        this.resetCardButtonClass = 'banned';
      }
    },
  },
  mounted() {
    this.InitSound();
    this.CreateCardList('Normal');
  },
  data() {
    let _this = this;
    return {
      dialogMsg: '',
      dialogClass: 'dialog_hide',
      resetCardButtonClass: '',
      sound_BGM: '',
      sound_SE: '',
      sound_Url: {
        monster: 'SE/ElementCard_Kill.wav',
        card: 'SE/ElementCard_TapCard.wav',
        draw: 'SE/ElementCard_Draw.wav',
        win: 'SE/ElementCard_Win.wav',
        lost: 'SE/ElementCard_Lost.wav',
        skill_AddHP: 'SE/ElementCard_AddHPMP.wav',
        skill_WhirlwindSword: 'SE/Battle_Attack_Blade.wav'
      },
      bgmState: '关',
      seState: '关',
      gameState: true,
      bossState: true,
      healthPoint: 10,
      actionPoint: 10,
      maxHealthPoint: 10,
      maxActionPoint: 10,
      monster_1_Dmg: 1,
      monster_2_Dmg: 2,
      monster_3_Dmg: 3,
      monster_Boss_Dmg: 5,
      gameScore: 0,
      allCardCount: 70,
      remainCardCount: 70,
      blackCardCount: 2,
      createdCardCount: 0,
      allClearCardCount: 0,
      cardList: [
        ['monsterCard_1', 12],
        ['monsterCard_2', 10],
        ['monsterCard_3', 8],
        ['monsterCard_Boss', 1],
        ['healthPointCard_1', 3],
        ['healthPointCard_2', 3],
        ['healthPointCard_3', 3],
        ['actionPointCard_1', 3],
        ['actionPointCard_2', 3],
        ['actionPointCard_3', 3],
        ['scoreCard_1', 10],
        ['scoreCard_2', 8],
        ['scoreCard_3', 3],
      ],
      curCardList: [],
      desktopCardList_Origin: [],
      desktopCardList_Change: [],
      skill: {
        skill_1(e) {
          if(_this.skill.skill_1_cd != 0) return;
          console.log('使用技能:旋风剑');
          _this.skill.skill_WhirlwindSword();
        },
        skill_2(e) {
          if(_this.skill.skill_2_cd != 0) return;
          console.log('使用技能:行动力恢复');
          _this.PlaySE(_this.sound_Url.skill_AddHP);
          _this.skill.skill_2_cd = 10;
          _this.CalcAP(5);
        },
        skill_3(e) {
          if(_this.skill.skill_3_cd != 0) return;
          console.log('使用技能:血量恢复');
          _this.PlaySE(_this.sound_Url.skill_AddHP);
          _this.skill.skill_3_cd = 15;
          _this.CalcHP(5);
        },
        skill_1_cd: 7,
        skill_2_cd: 7,
        skill_3_cd: 10,
        skill_WhirlwindSword_Ready: false,
        skill_WhirlwindSword(e) {
          if(!_this.skill.skill_WhirlwindSword_Ready)
          {
            _this.skill.skill_1_cd = 10;
            _this.skill.skill_WhirlwindSword_Ready = true;
          }
          else
          {
            _this.skill.skill_WhirlwindSword_Ready = false;
            _this.PlaySE(_this.sound_Url.skill_WhirlwindSword);
            //获得点击的元素
            e.target.classList.add('curclick');
            let desktopCardList = document.getElementById('desktop_background').getElementsByClassName('card');
            for(let i = 0; i < 10; i++)
            {
              let element = desktopCardList[i];
              element.classList.forEach((className) => {
                if(className == 'curclick')
                {
                  element.classList.remove('curclick');
                  if(i != 0 && i != 5)  //点击的卡牌不在最左侧时
                  {
                    if(_this.CheckMonster(_this.desktopCardList_Origin[i - 1])) //检查左侧卡片是不是怪物
                    {
                      _this.SetCardtoClicked(desktopCardList[i - 1]);
                    }
                  }
                  if(i != 4 && i != 9)  //点击的卡牌不在最右侧时
                  {
                    if(_this.CheckMonster(_this.desktopCardList_Origin[i + 1])) //检查右侧卡片是不是怪物
                    {
                      _this.SetCardtoClicked(desktopCardList[i + 1]);
                    }
                  }
                  if(i < 5)  //点击的卡牌在上侧时
                  {
                    if(_this.CheckMonster(_this.desktopCardList_Origin[i + 5])) //检查下方卡片是不是怪物
                    {
                      _this.SetCardtoClicked(desktopCardList[i + 5]);
                    }
                  }
                  else  //点击的卡牌在下侧
                  {
                    if(_this.CheckMonster(_this.desktopCardList_Origin[i - 5])) //检查上方卡片是不是怪物
                    {
                      _this.SetCardtoClicked(desktopCardList[i - 5]);
                    }
                  }
                }
              })
            }
          }
        },
      },
    }
  },
  methods: {
    InitSound() {
      console.log('初始化声音');
      //修改声音大小
      this.sound_BGM = document.getElementById('GameBGM');
      this.sound_BGM.volume = 0.5;
      this.sound_SE = document.getElementById('GameSE');
      this.sound_SE.volume = 0.5;
      //读取cookie保存的声音设置
      if(this.GetCookie('bgmState') == 'true')
      {
        this.SwitchBGMState();
      }
      if(this.GetCookie('seState') == 'true')
      {
        this.SwitchSEState();
      }
    },
    SwitchBGMState() {
      console.log('切换音乐开关');
      if(this.bgmState == '关')
      {
        this.bgmState = '开';
        document.cookie = 'bgmState = true';
      }
      else
      {
        this.bgmState = '关';
        document.cookie = 'bgmState = false';
      }
    },
    SwitchSEState() {
      console.log('切换音效开关');
      if(this.seState == '关')
      {
        this.seState = '开';
        document.cookie = 'seState = true';
      }
      else
      {
        this.seState = '关';
        document.cookie = 'seState = false';
      }
    },
    PlaySE(sound) {
      if(this.seState == '开')
      {
        this.sound_SE.setAttribute('src', sound);
        this.sound_SE.play();
      }
    },
    CreateCardList(mode) {
      if(mode == 'Normal')
      {
        //普通模式
        console.log('创建卡牌列表 - 普通模式');
        //将序号填满数组
        for(let p = 0; p < this.allCardCount; p++)
        {
          this.curCardList.push(p);
        }
        //打乱卡片列表
        this.RandomCardList();

        //将序号代表的卡片写入数组
        this.cardList.forEach((card) => {
          this.WriteCardLocation(card[0],card[1],this.createdCardCount);
        });

        //将卡片放入牌桌
        this.DrawCardtoDesktop(10);
      }
      else
      {
        //无尽模式
      }
    },
    RandomCardList() {
      console.log('打乱卡片列表');
      //将序号打乱并判断BOSS是不是在牌堆最后10张，否则重新打乱
      let loop = true;
      while(loop)
      {
        this.ResetCardList();
        this.ResetCardList();
        if(this.bossState)
        {
          for(let i = this.curCardList.length - 10; i < this.curCardList.length; i++)
          {
            if(this.curCardList[i] == 30 || this.curCardList[i] == 'monsterCard_Boss')
            {
              loop = false;
              break;
            }
          }
        }
        else
        {
          return;
        }
      }
    },
    WriteCardLocation(cardName, cardCount, curCreatedCardCount) {
      console.log('将卡片数据写入数组');
      //将卡片数据写入数组
      let endCount = cardCount + curCreatedCardCount;
      //console.log(cardName + "\n" + cardCount + "\n" + curCreatedCardCount + "\n" + endCount)
      for(let i = curCreatedCardCount;i < endCount; i++)
      {
        let curCount = 0;
        this.curCardList.forEach((card) => {
          if(card == i) //序号对应
          {
            this.curCardList[curCount] = cardName;
            this.createdCardCount++;
          }
          curCount++
        });
      }
    },
    CreateBlackCard(cardCount) {
      this.desktopCardList_Change = this.DeepCopy(this.desktopCardList_Origin);
      if(cardCount < 3) return;
      console.log('创建黑暗牌');
      for(let i = 0; i < this.blackCardCount; i++)
      {
        //随机一个位置
        let blackCardLocation = Math.floor(Math.random() * cardCount);
        //检测这个位置有没有被使用以及是不是BOSS
        if(this.desktopCardList_Change[blackCardLocation] == 'blackCard' || this.desktopCardList_Change[blackCardLocation] == 'monsterCard_Boss')
        {
          i--;
        }
        else
        {
          this.desktopCardList_Change[blackCardLocation] = 'blackCard';
        }
      }
    },
    RetrunSkillClass(cd) {
      if(cd)
      {
        return 'banned'
      }
      else
      {
        return '';
      }
    },
    ResetCardList() {
      this.curCardList.sort(() => {
          return Math.random() - 0.3;
      });
      //console.log(this.curCardList);
    },
    ReturnCardClassName(cardName) {
      if(cardName == 'blackCard')
      {
        return 'blackcard'
      }
    },
    ReturnImgClassName(cardName) {
      if(cardName.indexOf('monster') != -1)
      {
        //怪兽卡
        if(cardName.indexOf('Boss') != -1)
        {
          //BOSS
          return 'monstercard_boss';
        }
        else
        {
          //小怪
          if(cardName.indexOf('1') != -1)
          {
            return 'monstercard_0';
          }
          else if(cardName.indexOf('2') != -1)
          {
            return 'monstercard_1';
          }
          else if(cardName.indexOf('3') != -1)
          {
            return 'monstercard_2';
          }
        }
      }
      else if(cardName.indexOf('health') != -1)
      {
        //HP卡
        return 'hpcard';
      }
      else if(cardName.indexOf('action') != -1)
      {
        //AP卡
        return 'apcard';
      }
      else if(cardName.indexOf('score') != -1)
      {
        //分数卡
        return 'scorecard';
      }
    },
    ReturnPoint(cardName) {
      if(cardName.indexOf('monster') != -1)
      {
        //怪兽卡
        if(cardName.indexOf('Boss') != -1)
        {
          //BOSS
          return '5';
        }
        else
        {
          //小怪
          if(cardName.indexOf('1') != -1)
          {
            return '1';
          }
          else if(cardName.indexOf('2') != -1)
          {
            return '2';
          }
          else if(cardName.indexOf('3') != -1)
          {
            return '3';
          }
        }
      }
      else if(cardName.indexOf('health') != -1 || cardName.indexOf('action') != -1)
      {
        //HP和AP卡
        if(cardName.indexOf('1') != -1)
        {
          return '2';
        }
        else if(cardName.indexOf('2') != -1)
        {
          return '4';
        }
        else if(cardName.indexOf('3') != -1)
        {
          return '6';
        }
      }
      else if(cardName.indexOf('score') != -1)
      {
        //分数卡
        if(cardName.indexOf('1') != -1)
        {
          return '10';
        }
        else if(cardName.indexOf('2') != -1)
        {
          return '20';
        }
        else if(cardName.indexOf('3') != -1)
        {
          return '40';
        }
      }
    },
    ReturnText(cardName) {
      if(cardName.indexOf('monster') != -1)
      {
        //怪兽卡
        return '伤害';
      }
      else if(cardName.indexOf('health') != -1)
      {
        //HP卡
        return '生命恢复';
      }
      else if(cardName.indexOf('action') != -1)
      {
        //AP卡
        return '行动力恢复';
      }
      else if(cardName.indexOf('score') != -1)
      {
        //分数卡
        return '分数';
      }
    },
    ReturnCardBool(cardName) {
      if(cardName == 'blackCard')
      {
        return false;
      }
      else
      {
        return true;
      }
    },
    CheckMonster(cardName) {
      if(cardName.indexOf('monster') != -1 && cardName.indexOf('Boss') == -1)
      {
        if(cardName.indexOf('1') != -1)
        {
          this.CalcScore(10);
        }
        else if(cardName.indexOf('2') != -1)
        {
          this.CalcScore(20);
        }
        else if(cardName.indexOf('3') != -1)
        {
          this.CalcScore(40);
        }
        return true;
      }
      else
      {
        return false;
      }
    },
    ResetCard(ap = 0) {
      if(this.remainCardCount == 0) return;
      console.log('重新发牌');
      //消耗行动力
      this.CalcAP(ap);
      //清理卡牌数组的前十张卡牌
      this.curCardList.splice(0, 10);
      //获得当前牌桌上的所有牌
      let cards = document.getElementById('desktop_background').getElementsByClassName('card');
      //判断哪些是被点击的卡片并将其重新添加至卡牌数组
      let cardIndex = 0;
      Array.from(cards).forEach((card) => {
        let clearState = false;
        card.classList.forEach((className) => {
          if(className == 'clickedcard')
          {
            clearState = true;
            card.classList.remove('clickedcard');
          }
        });
        //未被点击的卡片
        if(!clearState)
        {
          //添加到卡牌数组内
          this.curCardList.push(this.desktopCardList_Origin[cardIndex]);
        }
        cardIndex++;
      });

      //将卡牌数组重新排序
      this.RandomCardList();
      //将卡牌放到牌桌
      let drawCount = 0;
      if(this.curCardList.length >= 10)
      {
        this.DrawCardtoDesktop(10);
        drawCount = 10;
      }
      else
      {
        this.DrawCardtoDesktop(this.curCardList.length);
        drawCount = this.curCardList.length;
      }
      //屏蔽多余的卡片
      for(let i = 9; i + 1 > drawCount; i--)
      {
        cards[i].classList.add('clickedcard');
      }
    },
    ClickCard(e, index) {
      if(!this.gameState)
      {
        return;
      }
      console.log('点击卡片');
      console.log('减少技能CD');
      this.skill.skill_1_cd--;
      this.skill.skill_2_cd--;
      this.skill.skill_3_cd--;
      let cardName = this.desktopCardList_Change[index] = this.desktopCardList_Origin[index];
      if(cardName.indexOf('monster') != -1) //怪兽卡
      {
        //播放音效
        this.PlaySE(this.sound_Url.monster);
        //减少行动力
        this.CalcAP(-1);
        if(cardName.indexOf('Boss') != -1)  //BOSS
        {
          this.MonsterBattle(4);
        }
        else  //小怪
        {
          if(this.skill.skill_WhirlwindSword_Ready)
          {
            this.skill.skill_WhirlwindSword(e);
          }
          if(cardName.indexOf('1') != -1)
          {
            this.MonsterBattle(1);
          }
          else if(cardName.indexOf('2') != -1)
          {
            this.MonsterBattle(2);
          }
          else if(cardName.indexOf('3') != -1)
          {
            this.MonsterBattle(3);
          }
        }
      }
      else if(cardName.indexOf('health') != -1) //HP卡
      {
        //播放音效
        this.PlaySE(this.sound_Url.card);
        if(cardName.indexOf('1') != -1)
        {
          this.CalcHP(2);
        }
        else if(cardName.indexOf('2') != -1)
        {
          this.CalcHP(4);
        }
        else if(cardName.indexOf('3') != -1)
        {
          this.CalcHP(6);
        }
      }
      else if(cardName.indexOf('action') != -1) //AP卡
      {
        //播放音效
        this.PlaySE(this.sound_Url.card);
        if(cardName.indexOf('1') != -1)
        {
          this.CalcAP(2);
        }
        else if(cardName.indexOf('2') != -1)
        {
          this.CalcAP(4);
        }
        else if(cardName.indexOf('3') != -1)
        {
          this.CalcAP(6);
        }
      }
      else if(cardName.indexOf('score') != -1)  //分数卡
      {
        //播放音效
        this.PlaySE(this.sound_Url.card);
        this.CalcAP(-1);
        if(cardName.indexOf('1') != -1)
        {
          this.CalcScore(10);
        }
        else if(cardName.indexOf('2') != -1)
        {
          this.CalcScore(20);
        }
        else if(cardName.indexOf('3') != -1)
        {
          this.CalcScore(40);
        }
      }
      //设置卡片为已点击状态
      this.SetCardtoClicked(e.target);
    },
    SetCardtoClicked(element) {
      setTimeout(()=>{
        element.classList.add('clickedcard');
        this.allClearCardCount++;
        //等待动画结束检查卡片状况
        setTimeout(() => {
          this.CheckDesktopCardStats();
        },500)
      },50);
    },
    CheckDesktopCardStats() {
      let clearCards = document.getElementById('desktop_background').getElementsByClassName('clickedcard');
      if(clearCards.length == 10)
      {
        if(this.allClearCardCount < 70)
        {
          //清理卡牌数组的前十张卡牌
          this.curCardList.splice(0, 10);
          //清理已经点击的卡牌的类
          let clearCount = 0
          Array.from(clearCards).forEach((card) => {
            if(clearCount < this.remainCardCount)
            {
              card.classList.remove('clickedcard');
              clearCount++;
            }
          });
          //发牌
          this.DrawCardtoDesktop(clearCount);
        }
        else
        {
          this.gameState = false;
          console.log('Win')
        }
      }
    },
    DrawCardtoDesktop(drawCount) {
      //播放SE
      this.PlaySE(this.sound_Url.draw);
      this.remainCardCount = this.curCardList.length;
      //将牌放进牌桌
      for(let i = 0; i < drawCount; i++)
      {
        if(this.curCardList[i] != undefined)
        {
          this.desktopCardList_Origin[i] = this.curCardList[i];
          //修改剩余卡牌数量
          this.remainCardCount--;
        }
      }
      //设置黑卡
      this.CreateBlackCard(drawCount);
    },
    MonsterBattle(type) {
      console.log('遭遇怪物');
      if(type == 1)       //Monster_1
      {
        this.CalcHP(-this.monster_1_Dmg, 10);
      }
      else if(type == 2)  //Monster_2
      {
        this.CalcHP(-this.monster_2_Dmg, 20);
      }
      else if(type == 3)  //Monster_3
      {
        this.CalcHP(-this.monster_3_Dmg, 40);
      }
      else if(type == 4)  //BOSS
      {
        this.CalcHP(-this.monster_Boss_Dmg, 100);
        this.bossState = false;
      }
    },
    CalcHP(hp, score = 0) {
      console.log('HP:' + hp);
      this.healthPoint += hp;
      if(this.CheckHP())
      {
        this.CalcScore(score);
        if(!this.healthPoint) //HP = 0
        {
          this.gameState = false;
          console.log('Game Over');
        }
      }
      else
      {
        this.healthPoint = 0;
        this.gameState = false;
        console.log('Game Over');
      }
    },
    CalcAP(ap) {
      if(this.actionPoint < -ap && ap <= 0)  //行动力不足
      {
        console.log('行动力不足');
        //计算当前行动力的差值
        let damage = -(-ap - this.actionPoint) * 2;
        //清空行动力
        this.actionPoint = 0;
        this.CalcHP(damage);
      }
      else
      {
        console.log('AP:' + ap);
        this.actionPoint += ap;
      }
    },
    CalcScore(score) {
      console.log('分数增加:' + score);
      this.gameScore += score;
    },
    CheckHP() {
      if(this.healthPoint < 0)
      {
        return false;
      }
      else
      {
        return true;
      }
    },
    DeepCopy(obj){
      let a=JSON.stringify(obj);
      let newObj=JSON.parse(a);
      return newObj;
    },
    GetCookie(name) {
      return document.cookie.match(`[;\s+]?${name}=([^;]*)`)?.pop();
    },
  },
}
</script>

<style>
  @font-face {
    font-family: 'tlt';
    src: url("../public/Font/tianliti.ttf");
  }

  html,
  body {
    position: absolute;
    width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
    font-family: 'tlt';
    user-select: none;
  }

  .dialog {
    position: absolute;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100vw;
    height: 100vh;
    background-color: rgba(0, 0, 0, 0.5);
    backdrop-filter: blur(0.5px);
    opacity: 0;
    z-index: 2;
  }

  .dialogUI {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 30em;
    height: 15em;
    background-color: rgb(206,197,195);
    border: rgb(75,67,114) 0.25em solid;
    border-radius: 2em;
    transition: 0.5s;
  }

  .dialogUI p {
    font-size: 4em;
    margin : 0.5em 0;
  }

  .dialogUI button {
    font-family: 'tlt';
    width: 8em;
    height: 2em;
    border-radius: 1em;
    font-size: 1.25em;
  }

  .dialog_show {
    pointer-events: auto;
    opacity: 1;
  }

  .dialog_hide {
    pointer-events: none;
    opacity: 0;
  }

  .Game {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100vw;
    height: 100vh;
  }

  .GameUI {
    display: flex;
    justify-content: center;
    width: 80em;
    height: 40em;
    background-color: rgb(69,59,87);
    border-radius: 2em;
  }

  .leftUI {
    display: flex;
    justify-content: center;
    width: 18.75%;
    min-width: 15em;
    height: 95%;
    margin: 1.25% 0;
    background-color: rgb(99,101,150);
    border: rgb(75,67,114) 0.125em solid;
    border-radius: 1em;
    box-sizing: border-box;
  }

  .leftUI_background {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 95%;
    height: 98%;
    margin: 2.8% 0;
    background-color: rgb(206,197,195);
    border-radius: 0.75em;
  }

  .exit {
    width: 90%;
    height: 2em;
    margin-top: 1em;
    border: rgb(127,130,166) 0.125em solid;
    border-radius: 1em;
    background-color: rgb(223,219,225);
    cursor: pointer;
  }

  .exit p {
    width: 100%;
    margin-top: 0.2em;
    text-align: center;
    font-size: 1.2em;
  }

  .resetcard {
    width: 12.5em;
    min-height: 6.35em;
    margin: 1em 0;
    background-color: rgb(144,124,88);
    border: rgb(123,100,78) 0.125em solid;
    border-radius: 1em;
    overflow: hidden;
    cursor: pointer;
  }

  .resetcard_background {
    width: 100%;
    height: 150%;
    padding-right: 6em;
    background-color: rgb(224,220,200);
    transform: rotate(255deg);
  }

  .resetcardUI {
    display: flex;
    position: absolute;
    width: 12.5em;
    height: 6.35em;
    margin-top: -9.5em;
  }

  .resetcardUI_left {
    width: 4.9em;
    height: 6.4em;
  }

  .cardback {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 3em;
    height: 4.5em;
    margin: 0.8em;
    background-color: rgb(139,128,175);
    border: rgb(121,114,153) 0.125em solid;
    border-radius: 0.5em;
    overflow: hidden;
  }

  .cardback_pattern {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 3em;
    height: 3em;
    background-color: rgb(194,188,214);
    border-radius: 0.2em;
    transform: rotate(-45deg);
  }

  .cardback_pattern p {
    font-size: 1.5em;
    transform: rotate(45deg);
  }

  .resetcardUI_right {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    margin-left: 0.85em;
  }

  .resetcardUI_righttop {
    margin: 0.5em 0;
  }

  .resetcardUI_righttop p,
  .resetcardUI_rightdown p{
    margin: 0;
    font-size: 1.25em;
  }

  .resetcardUI_rightdown {
    display: flex;
  }

  .resetcardUI_rightdown span {
    width: 2em;
    height: 2em;
    background-image: url('../public/IMG/APItem.png');
    background-repeat: no-repeat;
    background-size: cover;
  }

  .resetcardUI_rightdown p {
    margin-top: 0.1em;
  }

  .skilllist {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 90%;
  }

  .skilllist p {
    margin: 0;
    font-size: 1.2em;
  }

  .skill {
    width: 100%;
    height: 5em;
    margin-top: 1em;
    margin-bottom: 1em;
    background-color: rgb(139,128,175);
    border: rgb(121,114,153) 0.125em solid;
    border-radius: 1em;
    overflow: hidden;
    cursor: pointer;
  }

  .skill_background {
    width: 15em;
    height: 15em;
    margin-left: 3em;
    margin-top: -8em;
    background-color: rgb(228,221,226);
    border-radius: 100%;
  }

  .skill_UI {
    display: flex;
    align-items: center;
    width: 100%;
    height: 100%;
    margin-top: -7em;
    margin-left: 0.5em;
  }
  
  .skillimg {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 4em;
    height: 4em;
    border: 0.125em solid rgb(172,168,185);
    border-radius: 100%;
    background-repeat: no-repeat;
    background-size: cover;
    overflow: hidden;
  }

  .skillimg p {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
    margin: 0;
    background-color: rgba(0, 0, 0,0.5);
    color: whitesmoke;
    font-size: 2em;
    text-shadow: 0.125em 0.125em rgb(139,128,175);
  }

  .skill_1 {
    background-color: rgb(180,82,76);
    background-image: url('../public/IMG/Skill_Sword.PNG');
  }

  .skill_2 {
    background-color: rgb(120,158,109);
    background-image: url('../public/IMG/APSkill.png');
  }

  .skill_3 {
    background-color: rgb(120,158,109);
    background-image: url('../public/IMG/HPSkill.png');
  }

  .skillinfo {
    margin-left: 0.5em;
  }

  .rightUI {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 78.75%;
    height: 95%;
    margin: 1.25% 0;
    border-radius: 1em;
    box-sizing: border-box;
  }

  .statusbar {
    display: flex;
    flex-direction: row-reverse;
    align-items: center;
    width: 98.8%;
    height: 5%;
    margin: 0.8em 0;
    /* background-color: rgb(206,197,195); */
    border-radius: 1em;
  }

  .statusbar span {
    width: 2em;
    height: 2em;
    margin-left: -1em;
    background-repeat: no-repeat;
    background-size: cover;
  }

  .statusbar p {
    margin-left: 0.18em;
    color: rgb(227,225,252);
    text-shadow: 3px 3px rgb(81,81,97);
    font-size: 1.5em;
  }

  .buttongroup {
    display: flex;
    flex-direction: row-reverse;
    margin-right: 13.5%;
  }

  .buttongroup div {
    margin-left: 2.05em;
  }

  .bgmbutton {
    cursor: pointer;
  }

  .sebutton {
    margin-right: -1.3em;
    cursor: pointer;
  }

  .gamescore,
  .actionpoint,
  .healthpoint,
  .sebutton,
  .bgmbutton {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 9em;
    height: 2em;
    margin-left: 1em;
    background-color: rgb(125,101,138);
    border: rgb(83,70,99) 0.125em solid;
    border-radius: 2em;
  } 

  .gamescore span {
    background-image: url('../public/IMG/ScoreItem.png');
  }

  .actionpoint span {
    background-image: url('../public/IMG/APItem.png');
  }

  .healthpoint span {
    background-image: url('../public/IMG/HPItem.png');
  }

  .carddesktop {
    width: 100%;
    height: 95%;
    background-color: rgb(99,101,150);
    border: rgb(75,67,114) 0.125em solid;
    border-radius: 1em;
  }
  
  .desktop_background {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    width: 98.8%;
    min-width: 53.75em;
    height: 98%;
    margin: 0.5% 0.6%;
    background-color: rgb(206,197,195);
    border-radius: 0.75em;
    overflow: auto;
  }

  .blackcard {
    background-image: url('../public/IMG/BlackCard.png');
    animation: blackcard_anime 60s linear infinite;
  }

  @keyframes blackcard_anime {
    0% {
      background-position: 0px;
    }
    100% {
      background-position: 512px;
    }
  }

  .clickedcard {
    opacity: 0;
    pointer-events: none;
  }

  .card {
    width: 9em;
    height: 14em;
    margin: 1em;
    background-color: rgb(69,52,86);
    border: rgb(75,67,114) 0.125em solid;
    border-radius: 1em;
    overflow: hidden;
    cursor: pointer;
    transition: 0.5s;
  }

  .card_background {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 17em;
    height: 17em;
    margin-left: -4em;
    margin-top: -3.2em;
    background-color: rgb(228,221,226);
    border-radius: 100%;
    pointer-events: none;
  }

  .cardimg {
    width: 5em;
    height: 5em;
    margin-top: 25%;
    background-repeat: no-repeat;
    background-size: cover;
  }

  .cardpoint {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 1.5em;
    height: 1.5em;
    margin-top: 0.5em;
    font-size: 1.5em;
    background-color: rgb(194,188,214);
    border-radius: 100%;
  }

  .cardtext {
    margin-top: 0.5em;
    font-size: 1.5em;
  }

  .monstercard_0 {
    background-image: url('../public/IMG/Monster_0.png');
  }

  .monstercard_1 {
    background-image: url('../public/IMG/Monster_1.png');
  }

  .monstercard_2 {
    background-image: url('../public/IMG/Monster_2.png');
  }

  .monstercard_boss {
    background-image: url('../public/IMG/Monster_Boss.png');
  }

  .hpcard {
    background-image: url('../public/IMG/HPItem.png');
  }

  .apcard {
    background-image: url('../public/IMG/APItem.png');
  }

  .scorecard {
    background-image: url('../public/IMG/ScoreItem.png');
  }

  .banned {
    cursor: not-allowed;
  }
</style>