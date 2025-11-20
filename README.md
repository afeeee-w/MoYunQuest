# MoYunQuest
# The script of the game goes in this file.

## 设置游戏全局字体
define gui.text_font = "fonts/LXGWWenKai-Medium.ttf"
define gui.name_text_font = "fonts/LXGWWenKai-Medium.ttf" 
define gui.interface_text_font = "fonts/LXGWWenKai-Medium.ttf"
define gui.button_text_font = "fonts/LXGWWenKai-Medium.ttf"
define gui.choice_button_text_font = "fonts/LXGWWenKai-Medium.ttf"

## 定义角色
define mc = Character("我", color="#c8ffc8")  # 主角，浅绿色
define ml = Character("墨灵", color="#c8c8ff")  # 墨灵，浅紫色

## 定义一些特效和转场
define slowfade = Fade(1.5, 0.5, 1.5)  # 慢速淡入淡出
define poemfade = Fade(2.0, 1.0, 2.0)  # 诗句场景专用转场

## AI功能模块 - DeepSeek集成
init python:
    import random
    import sys
    import os
    
    # 尝试导入OpenAI库
    try:
        from openai import OpenAI
        openai_available = True
    except ImportError:
        openai_available = False
        print("OpenAI库未安装，将使用模拟模式")
    
    # DeepSeek API配置
    DEEPSEEK_API_KEY = "5eb49b01-ca2b-439b-a0de-9b1f39cc36e5"
    DEEPSEEK_BASE_URL = "https://api.deepseek.com/v1"
    
    def init_openai_client():
        """初始化DeepSeek客户端"""
        if not openai_available:
            return None
            
        try:
            client = OpenAI(
                api_key=DEEPSEEK_API_KEY,
                base_url=DEEPSEEK_BASE_URL
            )
            # 测试连接
            return client
        except Exception as e:
            print(f"DeepSeek客户端初始化失败: {e}")
            return None
    
    def ai_generate_poem(theme, existing_lines=""):
        """使用DeepSeek生成诗句"""
        # 初始化客户端
        client = init_openai_client()
        
        if client is None:
            return get_fallback_poem(theme, existing_lines)
        
        try:
            # 构建提示词
            if existing_lines:
                prompt = f"""你是一位精通中国古典诗词的专家。请根据以下上下文补全诗句：

{existing_lines}

主题：{theme}

要求：
1. 补全的诗句要符合古典诗词格律
2. 语言优美，意境深远
3. 与上下文衔接自然
4. 只输出补全的诗句，不要额外解释"""
            else:
                prompt = f"""你是一位精通中国古典诗词的专家。请以'{theme}'为主题创作一句中国古典风格的诗句。

要求：
1. 符合古诗格律（五言或七言）
2. 语言优美，意境深远
3. 体现中国传统文化韵味
4. 只输出诗句本身，不要额外解释"""
            
            # 调用DeepSeek API
            response = client.chat.completions.create(
                model="deepseek-chat",
                messages=[
                    {"role": "system", "content": "你是一位精通中国古典诗词的专家，擅长创作和补全古典诗句。"},
                    {"role": "user", "content": prompt}
                ],
                temperature=0.8,
                max_tokens=50
            )
            
            # 提取生成的文本
            poem_text = response.choices[0].message.content.strip()
            
            # 清理文本
            poem_text = clean_poem_text(poem_text)
            
            if poem_text and len(poem_text) > 2:
                return poem_text
            else:
                return get_fallback_poem(theme, existing_lines)
                
        except Exception as e:
            print(f"DeepSeek API调用失败: {e}")
            return get_fallback_poem(theme, existing_lines)
    
    def clean_poem_text(text):
        """清理AI返回的诗句文本"""
        # 移除可能的引号、特殊符号等
        text = text.replace('"', '').replace("'", "")
        text = text.replace("「", "").replace("」", "")
        text = text.replace("《", "").replace("》", "")
        text = text.replace("：", "：").replace(":", "：")
        
        # 移除常见的AI回应前缀
        prefixes = ["当然，", "好的，", "以下是", "我创作的诗句是：", "补全的诗句是："]
        for prefix in prefixes:
            if text.startswith(prefix):
                text = text[len(prefix):]
        
        text = text.strip()
        
        # 如果包含多个句子，取第一句
        if "。" in text:
            text = text.split("。")[0] + "。"
        elif "！" in text:
            text = text.split("！")[0] + "！"
        elif "？" in text:
            text = text.split("？")[0] + "？"
        elif "，" in text:
            parts = text.split("，")
            if len(parts) >= 2:
                text = parts[0] + "，" + parts[1]
        
        return text
    
    def get_fallback_poem(theme, existing_lines=""):
        """备用诗句库"""
        extended_poems = {
            "明月": ["举杯邀明月，对影成三人", "海上生明月，天涯共此时", "明月松间照，清泉石上流", "月出惊山鸟，时鸣春涧中"],
            "秋风": ["秋风清，秋月明", "洛阳城里见秋风，欲作家书意万重", "秋风萧瑟天气凉", "何处秋风至，萧萧送雁群"],
            "山水": ["行到水穷处，坐看云起时", "青山遮不住，毕竟东流去", "江流天地外，山色有无中", "水是眼波横，山是眉峰聚"],
            "离别": ["多情自古伤离别", "相见时难别亦难", "劝君更尽一杯酒，西出阳关无故人", "执手相看泪眼，竟无语凝噎"],
            "思乡": ["露从今夜白，月是故乡明", "故乡何处是，忘了除非醉", "烽火连三月，家书抵万金", "故园东望路漫漫，双袖龙钟泪不干"],
            "友情": ["桃花潭水深千尺，不及汪伦送我情", "海内存知己，天涯若比邻", "劝君更尽一杯酒，西出阳关无故人", "我寄愁心与明月，随君直到夜郎西"],
            "春天": ["春眠不觉晓，处处闻啼鸟", "好雨知时节，当春乃发生", "春色满园关不住，一枝红杏出墙来", "人面不知何处去，桃花依旧笑春风"],
            "夏天": ["小荷才露尖尖角，早有蜻蜓立上头", "接天莲叶无穷碧，映日荷花别样红", "稻花香里说丰年，听取蛙声一片", "力尽不知热，但惜夏日长"],
            "冬天": ["千山鸟飞绝，万径人踪灭", "忽如一夜春风来，千树万树梨花开", "孤舟蓑笠翁，独钓寒江雪", "梅须逊雪三分白，雪却输梅一段香"],
            "梅花": ["墙角数枝梅，凌寒独自开", "疏影横斜水清浅，暗香浮动月黄昏", "零落成泥碾作尘，只有香如故", "梅须逊雪三分白，雪却输梅一段香"],
            "竹子": ["咬定青山不放松，立根原在破岩中", "竹外桃花三两枝，春江水暖鸭先知", "斑竹枝，斑竹枝，泪痕点点寄相思", "新竹高于旧竹枝，全凭老干为扶持"],
            "江河": ["黄河之水天上来，奔流到海不复回", "无边落木萧萧下，不尽长江滚滚来", "大江东去，浪淘尽，千古风流人物", "问君能有几多愁，恰似一江春水向东流"],
            "雨": ["好雨知时节，当春乃发生", "夜来风雨声，花落知多少", "清明时节雨纷纷，路上行人欲断魂", "天街小雨润如酥，草色遥看近却无"],
            "雪": ["北国风光，千里冰封，万里雪飘", "窗含西岭千秋雪，门泊东吴万里船", "白雪却嫌春色晚，故穿庭树作飞花", "燕山雪花大如席，片片吹落轩辕台"],
            "酒": ["对酒当歌，人生几何", "葡萄美酒夜光杯，欲饮琵琶马上催", "今朝有酒今朝醉，明日愁来明日愁", "醉里挑灯看剑，梦回吹角连营"]
        }
        
        # 如果有现有诗句，尝试智能匹配
        if existing_lines:
            for key, poems in extended_poems.items():
                if any(keyword in existing_lines for keyword in [key, "青山", "白水", "戍鼓", "边秋"]):
                    return random.choice(poems)
        
        # 智能主题匹配
        theme_key = theme
        for key in extended_poems:
            if key in theme:
                theme_key = key
                break
        else:
            # 如果主题不在预设中，使用通用主题
            if any(word in theme for word in ["花", "草", "树", "木"]):
                theme_key = "春天"
            elif any(word in theme for word in ["热", "夏", "荷", "莲"]):
                theme_key = "夏天" 
            elif any(word in theme for word in ["冷", "冬", "雪", "梅"]):
                theme_key = "冬天"
            elif any(word in theme for word in ["河", "江", "海", "水"]):
                theme_key = "江河"
            else:
                theme_key = "自然"
        
        poems = extended_poems.get(theme_key, ["文思如泉涌，妙句自然成"])
        return random.choice(poems)

# 测试AI连接
init 1 python:
    if openai_available:
        print("正在测试DeepSeek连接...")
        test_client = init_openai_client()
        if test_client:
            print("✅ DeepSeek连接成功！")
        else:
            print("❌ DeepSeek连接失败，将使用模拟模式")
    else:
        print("❌ OpenAI库未安装，将使用模拟模式")

# 游戏开始
label start:
    ## 开场 - 古籍社室内
    scene bg_old_library with slowfade
    ## 给玩家时间感受环境
    "深夜的古籍社活动室，月光从窗棂洒入，尘埃在光柱中缓缓飘舞。"
    
    ## 播放背景音乐
    play music "audio/main_theme.mp3" volume 0.2 fadein 3.0
    
    ## 开场对话开始
    mc "唉…古籍社的招新又失败了。这年头，还有谁会对这些发黄的旧书感兴趣呢？"
    
    mc "毕业前要是再招不到新人，这个社团恐怕就要被注销了…"
    
    ## 切换到书架特写，增强发现感
    scene bg_library_shelf with dissolve
    mc "嗯？书架最顶层……好像有一本我从没见过的书。"
    
    mc "你踮起脚，费力地将那本蒙着厚厚灰尘的线装书取了下来。"
    
    ## 切回室内全景
    scene bg_old_library with dissolve
    mc "奇怪，书封上没有书名，只有一些像是水渍留下的……墨色纹路？"
    
    mc "你下意识地用手拂去封面的灰尘。就在你的指尖触碰到书页的瞬间——"
    
    ## 可以在这里添加一个音效
    play sound "audio/glow.mp3"
    
    ## 显示墨灵立绘，同时背景稍微变化
    show ml_normal at center with dissolve
    "书页间泛起微光，墨色纹路仿佛活了过来，在空气中流动。"
    
    mc "！刚才是……眼花了吗？"
    
    ml "……终于……有人……能触碰到'墨缘'了么？"
    
    ml "沉眠百载，幸得君唤。我名'墨灵'，乃此书之魂。"
    
    ## 背景出现微妙变化，暗示魔法效果
    scene bg_old_library_glow with dissolve
    show ml_normal at center
    "周围的空气似乎变得不同，古老的书架仿佛笼罩在一层淡淡的微光中。"
    
    ml "看来，你我有缘..."

    ## 第一个选择分支
    menu:
        ml "你是否愿意助我修复这书中残缺的诗文，解开往昔的谜团？"
        "我愿意帮忙！这听起来很有趣。":
            jump willing_to_help
        "我...我有点害怕。这太奇怪了。":
            jump afraid_to_help
        "修复诗文？我需要先知道更多细节。":
            jump ask_for_details

## 分支一：愿意帮忙
label willing_to_help:
    mc "我愿意帮忙！这听起来很有趣。"
    ml "太好了！有缘人果然心怀热忱。"
    
    ## 切换到更具魔法感的背景
    scene bg_library_magic with dissolve
    show ml_normal at center
    "墨灵轻轻挥手，周围的书籍仿佛被注入了生命，书页无风自动。"
    
    ml "让我们开始第一首诗的修复吧..."
    jump first_poem_puzzle

## 分支二：害怕拒绝
label afraid_to_help:
    mc "我...我有点害怕。这太奇怪了。"
    ml "（轻声叹息）无妨...机缘未到，不可强求。"
    
    ## 切换到冷色调背景，增强拒绝的寂寥感
    scene bg_old_library_cold with fade
    show ml_normal at center
    "室内的光芒渐渐暗淡，墨灵的身影开始变得透明。"
    
    ml "若你改变心意，我仍在此书中等待。"
    "墨灵的身影逐渐淡去，古籍恢复了普通的样子。"
    
    ## 停留在寂寥的场景中一会儿
    scene bg_old_library_cold with slowfade
    "活动室恢复了往常的寂静，只有月光依旧洒在书架上。"
    "但那本古书，似乎还在等待着什么..."
    
    ## 坏结局或暂时结束
    return

## 分支三：询问细节
label ask_for_details:
    mc "修复诗文？我需要先知道更多细节。"
    
    ## 切换到更亲密的对话场景
    scene bg_library_close with dissolve
    show ml_normal at center
    
    ml "此书收录了历代文人诗稿，但时光流转，许多诗句已然残缺。"
    "墨灵指向空中，几行发光的文字缓缓浮现。"
    
    ml "我需要借助你的感知，与我一同补全这些失落的文字。"
    ml "每修复一首诗，就能解开一段被遗忘的故事。"
    
    ## 再次给出选择
    menu:
        ml "现在，你愿意帮助我吗？"
        
        "好吧，我答应你。":
            jump willing_to_help
            
        "抱歉，我还是无法接受。":
            jump afraid_to_help


## === 增强版第一首诗修复（诗句播放时隐藏墨灵）===
label first_poem_puzzle:
    ## 场景一：诗词意境引入
    scene bg_riverside with poemfade
    
    "周围的景象缓缓变化，你发现自己站在一条清澈的河边。"
    "远处青山如黛，近处白水环绕，微风拂过水面，泛起粼粼波光。"
    
    mc "这是...诗中的景象？"
    
    show ml_normal at center with dissolve
    ml "正是。让我们沉浸在这意境中，感受诗人当时的心境。"
    pause 2.0
    
    ## 场景二：第一联诗句展示（隐藏墨灵）
    hide ml_normal with dissolve
    "晨曦洒在河面上，远山横亘在北边，清澈的河水绕着东城流淌。"
    
    show expression Text("「青山横北郭，白水绕东城。」", size=36, color="#E8D5B7") as line1 with dissolve:
        yalign 0.3
        xalign 0.5
    pause 3.0
    
    "看这青山白水，正是送别的绝佳地点。"
    "你能感受到诗人与友人即将分别时的那份不舍。"
    pause 2.0
    
    hide line1 with dissolve
    
    ## 场景三：缺失句子的意境铺垫
    "一条古道延伸向远方，路边的孤蓬在风中摇曳。"
    "此情此景，让你不禁思考下一句该如何接续..."
    
    show expression Text("「______________」", size=36, color="#FF6B6B") as missing_line with dissolve:
        yalign 0.3
        xalign 0.5
    pause 2.5
    
    show ml_normal at center with dissolve
    ml "友人即将远行，如这孤蓬般漂泊万里..."
    pause 1.5
    hide ml_normal with dissolve
    
    hide missing_line with dissolve
    
    ## 场景四：第三联诗句展示（隐藏墨灵）
    "夕阳西下，两位友人在马旁挥手告别。"
    "马儿似乎也感受到了离愁，发出萧萧的嘶鸣。"
    
    show expression Text("「挥手自兹去，萧萧班马鸣。」", size=36, color="#E8D5B7") as line3 with dissolve:
        yalign 0.3
        xalign 0.5
    pause 3.0
    
    show ml_normal at center with dissolve
    ml "感受到这离别之情了吗？现在，请补全中间的诗句。"
    pause 1.5
    
    hide line3 with dissolve
    
    ## 切换到选择场景
    scene bg_riverside with dissolve
    show ml_normal at center
    
    menu:
        "请选择补全的诗句："
        
        "此地一为别，孤蓬万里征。":
            jump correct_poem_choice
            
        "浮云游子意，落日故人情。":
            ml "（微笑）这句诗很美，但它是下一联的。"
            jump wrong_poem_choice
            
        "桃花潭水深千尺，不及汪伦送我情。":
            ml "这是李白的另一首赠别诗呢。"
            jump wrong_poem_choice
            
        "海内存知己，天涯若比邻。":
            ml "意境相近，但非此诗原句。"
            jump wrong_poem_choice

## 正确选择 - 增强版（诗句播放时隐藏墨灵）
label correct_poem_choice:
    ## 修复成功的完整场景体验
    scene bg_poem_success with hpunch
    
    show ml_normal at center with dissolve
    ml "太好了！正是「此地一为别，孤蓬万里征」！"
    
    "完整的诗句在空中浮现，与眼前的景色完美融合："
    
    hide ml_normal with dissolve
    
    show expression Text("「青山横北郭，白水绕东城。」", size=28, color="#E8D5B7") as line1 with dissolve:
        yalign 0.2
        xalign 0.5
    pause 1.5
    
    show expression Text("「此地一为别，孤蓬万里征。」", size=32, color="#FFD700") as line2 with dissolve:
        yalign 0.3
        xalign 0.5
    pause 2.0
    
    show expression Text("「挥手自兹去，萧萧班马鸣。」", size=28, color="#E8D5B7") as line3 with dissolve:
        yalign 0.4
        xalign 0.5
    pause 2.5
    
    ## 修复成功的特效
    play sound "audio/success.mp3"
    show expression Text("修复成功！", size=40, color="#fff") as text_effect with dissolve
    pause 2.0
    
    "周围的景色绽放出温暖的光芒，仿佛整首诗都活了过来。"
    "你能真切地感受到李白与友人分别时的那份深情。"
    pause 2.0
    
    hide line1
    hide line2
    hide line3
    hide text_effect with dissolve
    
    show ml_normal at center with dissolve
    ml "你已修复了第一首诗，真切地感受到了诗人李白与友人的离别之情。"
    
    ml "这首诗背后，是两位诗人的深厚友谊..."
    

## === 增强版第二首诗修复（诗句播放时隐藏墨灵）===
label second_poem_puzzle:
    ## 场景一：秋夜意境引入
    scene bg_autumn_night with poemfade
    
    "秋夜的凉意透过时空传来，戍楼的鼓声在寂静中格外清晰。"
    "一只孤雁的哀鸣划破夜空，更添几分凄凉。"
    
    mc "这秋夜...让人心生寂寥。"
    
    show ml_normal at center with dissolve
    ml "正是。让我们感受这战乱时期的秋夜思乡之情。"
    pause 2.0
    
    ## 场景二：第一联诗句展示（隐藏墨灵）
    hide ml_normal with dissolve
    "戍楼的轮廓在夜色中若隐若现，道路上已无行人。"
    
    show expression Text("「戍鼓断人行，边秋一雁声。」", size=36, color="#B8D0D8") as line1 with dissolve:
        yalign 0.3
        xalign 0.5
    pause 3.0
    
    "战乱时期的边关秋夜，连雁鸣都带着哀愁。"
    "你能感受到诗人在战乱中对亲人的深深牵挂。"
    pause 2.0
    
    hide line1 with dissolve
    
    ## 场景三：缺失句子的意境铺垫
    "寒露在草叶上凝结，月光洒在大地上，显得格外清冷。"
    "望着这轮明月，你想起了远方的故乡..."
    
    show expression Text("「______________」", size=36, color="#FF6B6B") as missing_line with dissolve:
        yalign 0.3
        xalign 0.5
    pause 2.5
    
    show ml_normal at center with dissolve
    ml "白露为霜，明月思乡...此刻你最想念的是什么？"
    pause 1.5
    hide ml_normal with dissolve
    
    hide missing_line with dissolve
    
    ## 场景四：第三联诗句展示（隐藏墨灵）
    "战火的阴影笼罩着大地，家书难达，战事未休。"
    "在这动荡的时代，对亲人的思念愈发强烈。"
    
    show expression Text("「寄书长不达，况乃未休兵。」", size=36, color="#B8D0D8") as line3 with dissolve:
        yalign 0.3
        xalign 0.5
    pause 3.0
    
    show ml_normal at center with dissolve
    ml "感受到这乱世中的思乡之情了吗？请补全中间的诗句。"
    pause 1.5
    
    hide line3 with dissolve
    
    ## 切换到选择场景
    scene bg_autumn_night with dissolve
    show ml_normal at center
    
    menu:
        "请选择补全的诗句："
        
        "露从今夜白，月是故乡明。":
            jump second_poem_correct
            
        "举头望明月，低头思故乡。":
            ml "这是李白的《静夜思》，意境相似但非此诗。"
            jump second_poem_wrong
            
        "床前明月光，疑是地上霜。":
            ml "这也是《静夜思》的开头，请再想想。"
            jump second_poem_wrong
            
        "明月松间照，清泉石上流。":
            ml "这是王维的山水诗，与思乡主题不符。"
            jump second_poem_wrong

label second_poem_correct:
    ## 修复成功的完整场景体验（诗句播放时隐藏墨灵）
    scene bg_poem_success2 with hpunch
    
    show ml_normal at center with dissolve
    ml "正确！正是「露从今夜白，月是故乡明」！"
    
    "完整的诗句在空中浮现，与秋夜景致相得益彰："
    
    hide ml_normal with dissolve
    
    show expression Text("「戍鼓断人行，边秋一雁声。」", size=28, color="#B8D0D8") as line1 with dissolve:
        yalign 0.2
        xalign 0.5
    pause 1.5
    
    show expression Text("「露从今夜白，月是故乡明。」", size=32, color="#FFD700") as line2 with dissolve:
        yalign 0.3
        xalign 0.5
    pause 2.0
    
    show expression Text("「寄书长不达，况乃未休兵。」", size=28, color="#B8D0D8") as line3 with dissolve:
        yalign 0.4
        xalign 0.5
    pause 2.5
    
    ## 修复成功的特效
    play sound "audio/success.mp3"
    show expression Text("修复成功！", size=40, color="#fff") as text_effect with dissolve
    pause 2.0
    
    "秋夜的寒意被温暖的光芒驱散，思乡之情化为深深的慰藉。"
    "你能真切地感受到杜甫在战乱中对弟弟的深切思念。"
    pause 2.0
    
    hide line1
    hide line2
    hide line3
    hide text_effect with dissolve
    
    show ml_normal at center with dissolve
    ml "你已修复第二首诗..."
    
    ## 跳转到AI创作体验
    ml "现在，你想尝试用新技术创作属于自己的诗句吗？"
    
    menu:
        "我想尝试AI创作！":
            jump ai_poem_creation
            
        "今天先到这里":
            jump traditional_ending

## 以下部分保持不变（故事揭晓、AI场景、结局等）
## ... [保持原有的 story_reveal_1, story_reveal_2, ai_poem_creation 等标签]

## 第二个故事揭晓
label story_reveal_2:
    scene bg_war_memory with slowfade
    show ml_normal at center
    
    ml "这首诗是杜甫的《月夜忆舍弟》，写于战乱时期..."
    
    ## 切换到战乱背景
    scene bg_war_time with dissolve
    show ml_normal at center
    ml "白露时节的寒凉，故乡明月的清辉，都寄托着诗人对弟弟的深切思念。"
    
    "战火的阴影与明月的清辉形成鲜明对比，你能感受到杜甫在乱世中的牵挂。"
    
    mc "在战乱中思念亲人...这种情感真的很沉重。"
    
    ## 回到温暖的结束场景
    scene bg_library_warm with slowfade
    show ml_normal at center
    ml "正是这些真挚的情感，让这些诗篇穿越千年，依然动人。"
    
    jump traditional_ending

## === 增强版AI诗句创作场景 ===
label ai_poem_creation:
    scene bg_library_magic with slowfade
    show ml_normal at center
    
    ml "现在，让我们尝试一些新的创作。"
    ml "你可以选择任何主题，我将借助天地文气，为你生成相关的诗句。"
    
    # 显示主题建议
    "墨灵挥手，空中浮现出一些创作主题的建议："
    show expression Text("建议主题：明月、秋风、山水、离别、思乡、友情\n春天、夏天、冬天、梅花、竹子、江河、雨、雪、酒", size=24, color="#E8D5B7") as theme_suggestions with dissolve:
        yalign 0.2
        xalign 0.5
    pause 2.5
    hide theme_suggestions with dissolve
    
    # 获取玩家输入的主题 - 更开放的提示
    $ user_theme = renpy.input("请输入任何你想创作的主题（如：明月、秋风、思乡、梅花、雨雪等）：", length=30)
    $ user_theme = user_theme.strip()
    
    if not user_theme:
        $ user_theme = "自然"
    
    mc "我想以[user_theme]为主题创作诗句。"
    
    ml "很好，让我凝聚文思，搜索与[user_theme]相关的诗意..."
    
    # 显示等待动画
    show ml_normal at center:
        linear 0.5 yalign 0.48
        linear 0.5 yalign 0.5
        repeat
    
    "墨灵眼中的墨色漩涡加速旋转，周围的书页无风自动，仿佛在搜索千年的诗库..."
    
    # 调用AI生成诗句 - 使用增强版函数
    $ ai_poem = ai_generate_poem(user_theme)
    
    # 停止动画
    show ml_normal at center
    
    ml "文思已聚，请听此句："
    
    # 显示AI生成的诗句（有特效）
    show expression Text("「" + ai_poem + "」", size=36, color="#FFD700") as ai_text with dissolve:
        yalign 0.3
        xalign 0.5
    
    pause 2.5
    
    mc "真是美妙的诗句！充满了[user_theme]的意境。"
    
    ml "这便是技术赋能文化的魅力——让古老的诗歌艺术焕发新的生机。"
    ml "通过AI的力量，我们可以探索更多诗歌的可能性，不受传统主题的限制。"
    
    hide ai_text with dissolve
    
    # 选择是否继续
    menu:
        ml "要再尝试其他主题吗？"
        
        "再试一次，换个主题":
            jump ai_poem_creation
            
        "尝试补全一首诗":
            jump ai_poem_completion
            
        "暂时到此为止":
            ml "希望这次体验让你感受到传统文化与AI结合的可能性。"
            jump ai_ending

## === AI诗句补全场景 ===
label ai_poem_completion:
    scene bg_library_magic with slowfade
    show ml_normal at center
    
    ml "现在让我们尝试补全一首诗。"
    ml "你可以提供一些已有的诗句，我将为你补全缺失的部分。"
    
    # 获取玩家输入的诗句
    $ existing_lines = renpy.input("请输入已有的诗句或上下文：", length=100)
    $ existing_lines = existing_lines.strip()
    
    if not existing_lines:
        $ existing_lines = "床前明月光"
    
    # 获取补全主题
    $ completion_theme = renpy.input("这首诗的主题是什么？（如：思乡、夜景等）：", length=20)
    $ completion_theme = completion_theme.strip()
    
    if not completion_theme:
        $ completion_theme = "自然"
    
    mc "我想补全这首诗：'[existing_lines]'，主题是[completion_theme]。"
    
    ml "很好，让我理解诗意，为你补全..."
    
    # 显示等待动画
    show ml_normal at center:
        linear 0.5 yalign 0.48
        linear 0.5 yalign 0.5
        repeat
    
    "墨灵闭目凝神，墨色文字在空中流转，仿佛在理解诗意的脉络..."
    
    # 调用AI补全诗句
    $ completed_poem = ai_generate_poem(completion_theme, existing_lines)
    
    # 停止动画
    show ml_normal at center
    
    ml "诗意已明，请听补全之句："
    
    # 显示补全的诗句
    show expression Text("「" + completed_poem + "」", size=36, color="#FFD700") as completed_text with dissolve:
        yalign 0.3
        xalign 0.5
    
    pause 2.5
    
    mc "这句补得真妙！与前面的诗句相得益彰。"
    
    ml "AI不仅能创作新诗，还能理解古典诗意，进行恰当的补全。"
    ml "这便是科技与传统文化的完美融合。"
    
    hide completed_text with dissolve
    
    # 选择下一步
    menu:
        ml "接下来你想做什么？"
        
        "再补全一首诗":
            jump ai_poem_completion
            
        "创作新诗句":
            jump ai_poem_creation
            
        "结束AI创作":
            ml "希望这些创作让你感受到诗歌的无限可能。"
            jump ai_ending

## 其他部分保持不变...
## ... [保持原有的故事揭晓、结局等部分]

## === AI体验结束 ===
label ai_ending:
    scene bg_library_warm with slowfade
    show ml_normal at center
    
    ml "今日的诗文修复与创作已告一段落。"
    ml "你不仅修复了古人的智慧，还通过新技术参与了诗歌的创作与探索。"
    
    mc "这真是奇妙的体验！AI让诗歌创作有了更多可能性。"
    
    ml "正是。文化需要传承，也需要创新。"
    ml "AI不是要取代传统，而是为我们提供新的视角和工具。"
    
    "墨灵的身影在温暖的光芒中渐渐变得透明。"
    
    ml "期待与你的下一次相遇，探索更多诗意的可能..."
    
    scene bg_old_library_changed with slowfade
    "古籍社恢复了平静，但那本古书似乎散发着科技与传统文化交融的光芒。"
    "你知道，这只是探索的开始..."
    
    ## 显示制作人员（更新版）
    show expression Text("《墨韵奇谈》\n\nAI技术支持：百度千帆ERNIE\n传统与科技的完美融合\n诗歌创作的新可能", size=28, color="#FFFFFF") as credits with dissolve
    pause 3.0
    hide credits with dissolve
    
    return


## 传统结局
label traditional_ending:
    scene bg_library_warm with slowfade
    show ml_normal at center
    
    ml "两首诗已修复，但书中仍有无数故事等待发掘..."
    ml "今日暂且到此，感谢你让这些被遗忘的文字重获新生。"
    
    "墨灵的身影在温暖的光芒中渐渐消散，但那感动仍留在你心中。"
    
    ## 最终回到现实，但有微妙变化
    scene bg_old_library_changed with slowfade
    "古籍社的灯光依旧，但你感觉这个世界似乎多了一些不一样的色彩。"
    "那本古书静静地躺在桌上，等待着下一次的相遇。"
    
    ## 游戏结束
    return

## 临时结束
label temporary_end:
    scene bg_old_library with slowfade
    show ml_normal at center
    
    ml "当你准备好时，随时可以回来继续修复诗文。"
    ml "这些被遗忘的故事，正等待着有缘人来唤醒..."
    
    "墨灵的身影渐渐淡去，古籍恢复了平静。"
    
    ## 停留在场景中，给玩家回味的时间
    scene bg_old_library with slowfade
    "月光依旧，但你知道，这个世界已经不一样了。"
    
    ## 游戏结束
    return
