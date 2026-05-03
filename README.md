# studentRp 学生报告生成脚本
from PIL import Image, ImageDraw, ImageFont
import os
import pandas as pd
import glob

# -------------------------- 1. 配置参数 --------------------------
# 橘红色系配色方案
IMAGE_WIDTH = 2480  # 报告宽度固定
BACKGROUND_COLOR = (255, 248, 240)  # 浅橘红背景
TITLE_BAR_COLOR = (230, 80, 30)  # 深橘红标题栏
TEXT_COLOR_MAIN = (80, 30, 15)  # 深棕红正文
TEXT_COLOR_SUB = (210, 70, 25)  # 橘红副标题
BORDER_COLOR = (255, 200, 170)  # 浅橘红边框

# 地区梯队
TIER1_REGIONS = {'北京市', '上海市', '广东省', '浙江省', '江苏省',
                 '四川省', '重庆市', '福建省', '安徽省'}
TIER2_REGIONS = {'湖北省', '山东省', '天津市', '河北省', '吉林省',
                 '江西省', '湖南省'}

# 字体配置
try:
    FONT_TITLE = ImageFont.truetype("C:/Windows/Fonts/msyhbd.ttc", 80)  # 标题粗体
    FONT_SUBTITLE = ImageFont.truetype("C:/Windows/Fonts/msyhbd.ttc", 50)  # 副标题（略小）
    FONT_HEADER = ImageFont.truetype("C:/Windows/Fonts/msyhbd.ttc", 50)  # 板块标题
    FONT_CONTENT = ImageFont.truetype("C:/Windows/Fonts/msyh.ttc", 36)  # 正文
    FONT_SMALL = ImageFont.truetype("C:/Windows/Fonts/msyh.ttc", 30)  # 小字
except:
    FONT_TITLE = FONT_HEADER = FONT_CONTENT = FONT_SMALL = ImageFont.load_default()
    FONT_SUBTITLE = FONT_SMALL  # 副标题默认使用小字
    print("警告：未找到指定字体，使用默认字体")

# 布局配置
MARGIN = 80  # 整体边距
LINE_SPACING = 30  # 行间距
BLOCK_SPACING = 60  # 板块间距
HEADER_SPACING = 30  # 标题与内容间距
BORDER_PADDING = 20  # 边框内边距
IMAGE_GAP = 40  # 图片间距
TITLE_LINE_SPACING = 20  # 标题两行之间的间距

# 图片配置
SUPPORTED_IMG_FORMATS = ['*.png', '*.jpg', '*.jpeg', '*.bmp']
IMG_SECTION_TITLE = "六、当地资讯参考"  # 第六部分标题
SINGLE_IMG_MAX_WIDTH_RATIO = 0.8  # 单张图片最大宽度比例
DOUBLE_IMG_MAX_WIDTH_RATIO = 0.42  # 两张图片并排时单张最大宽度比例
CURRENT_DIR = os.path.abspath(os.path.curdir)


# -------------------------- 2. 工具函数：文本换行绘制 --------------------------
def draw_wrapped_text(draw, text, x, y, max_width, font, color):
    """长文本自动换行绘制，返回绘制后的y坐标"""
    lines = []
    current_line = ""
    for char in text:
        test_line = current_line + char
        if draw.textlength(test_line, font=font) > max_width and current_line:
            lines.append(current_line)
            current_line = char
        else:
            current_line = test_line
    if current_line:
        lines.append(current_line)

    current_y = y
    for line in lines:
        draw.text((x, current_y), line, fill=color, font=font)
        current_y += font.size + LINE_SPACING
    return current_y


# -------------------------- 3. 图片匹配函数 --------------------------
def get_region_prefix(region):
    """提取地区名称前两字作为匹配关键词"""
    clean_region = region.strip()
    return clean_region[:2] if len(clean_region) >= 2 else clean_region


def match_region_image(region_prefix):
    """在同级目录搜索匹配的地区图片，返回图片路径（无匹配则返回None）"""
    if not region_prefix:
        print(f"⚠️ 地区关键词为空，无法匹配图片")
        return None

    print(f"\n🔍 开始匹配【{region_prefix}】相关图片：")
    print(f"   - 搜索目录：{os.path.basename(CURRENT_DIR)}")
    print(f"   - 匹配规则：文件名包含“{region_prefix}”（不区分大小写）")
    print(f"   - 支持格式：{', '.join(SUPPORTED_IMG_FORMATS)}")

    matched_imgs = []
    all_imgs = []
    for img_format in SUPPORTED_IMG_FORMATS:
        img_paths = glob.glob(os.path.join(CURRENT_DIR, img_format))
        for img_path in img_paths:
            if os.path.dirname(os.path.abspath(img_path)) != CURRENT_DIR:
                continue
            img_filename = os.path.basename(img_path)
            all_imgs.append(img_filename)
            img_name_no_suffix = os.path.splitext(img_filename)[0].lower()
            if region_prefix.lower() in img_name_no_suffix:
                matched_imgs.append(img_path)

    if all_imgs:
        print(f"   - 同级目录找到的所有图片：{all_imgs}")
    else:
        print(f"   - 同级目录未找到任何支持格式的图片")

    if matched_imgs:
        selected_img = matched_imgs[0]
        print(f"✅ 匹配成功！选中图片：{os.path.basename(selected_img)}")
        return selected_img
    else:
        print(f"⚠️ 未找到文件名含“{region_prefix}”的图片")
        return None


def match_illustration_image():
    """专门搜索名为“插图”的图片（精确匹配）"""
    print(f"\n🔍 开始匹配【插图】图片：")
    target_name = "插图"
    for img_format in SUPPORTED_IMG_FORMATS:
        img_paths = glob.glob(os.path.join(CURRENT_DIR, img_format))
        for img_path in img_paths:
            img_filename = os.path.basename(img_path)
            img_name_no_suffix = os.path.splitext(img_filename)[0].lower()
            if img_name_no_suffix == target_name.lower():
                print(f"✅ 找到插图：{img_filename}")
                return img_path
    print(f"⚠️ 未找到名为“插图”的图片（支持格式：{', '.join(SUPPORTED_IMG_FORMATS)}）")
    return None


def match_illustration1_image():
    """专门搜索名为“插图1”的图片（精确匹配）"""
    print(f"\n🔍 开始匹配【插图1】图片：")
    target_name = "插图1"
    for img_format in SUPPORTED_IMG_FORMATS:
        img_paths = glob.glob(os.path.join(CURRENT_DIR, img_format))
        for img_path in img_paths:
            img_filename = os.path.basename(img_path)
            img_name_no_suffix = os.path.splitext(img_filename)[0].lower()
            if img_name_no_suffix == target_name.lower():
                print(f"✅ 找到插图1：{img_filename}")
                return img_path
    print(f"⚠️ 未找到名为“插图1”的图片（支持格式：{', '.join(SUPPORTED_IMG_FORMATS)}）")
    return None


def match_illustration2_image():
    """专门搜索名为“插图2”的图片（精确匹配）"""
    print(f"\n🔍 开始匹配【插图2】图片：")
    target_name = "插图2"
    for img_format in SUPPORTED_IMG_FORMATS:
        img_paths = glob.glob(os.path.join(CURRENT_DIR, img_format))
        for img_path in img_paths:
            img_filename = os.path.basename(img_path)
            img_name_no_suffix = os.path.splitext(img_filename)[0].lower()
            if img_name_no_suffix == target_name.lower():
                print(f"✅ 找到插图2：{img_filename}")
                return img_path
    print(f"⚠️ 未找到名为“插图2”的图片（支持格式：{', '.join(SUPPORTED_IMG_FORMATS)}）")
    return None


def has_matched_images(region):
    """判断是否存在匹配的图片（用于预计算板块高度）"""
    region_img = match_region_image(get_region_prefix(region))
    illustration_img = match_illustration_image()
    illustration1_img = match_illustration1_image()
    illustration2_img = match_illustration2_image()
    return {
        'has_region_img': region_img is not None,
        'has_illustration_img': illustration_img is not None,
        'has_illustration1_img': illustration1_img is not None,
        'has_illustration2_img': illustration2_img is not None
    }


# -------------------------- 4. 图片绘制函数 --------------------------
def draw_images_section(img, draw, start_y, region):
    """绘制当地资讯参考图片（地区图片+插图2），返回结束y坐标"""
    block_start_y = start_y
    current_y = start_y

    # 绘制图片板块标题
    draw.text((MARGIN, current_y), IMG_SECTION_TITLE, fill=TEXT_COLOR_SUB, font=FONT_HEADER)
    current_y += FONT_HEADER.size + HEADER_SPACING

    # 获取两张图片：地区图片和插图2（固定）
    region_prefix = get_region_prefix(region)
    region_img_path = match_region_image(region_prefix)
    illustration2_img_path = match_illustration2_image()  # 固定使用插图2

    # 检查是否有图片
    has_region = region_img_path is not None
    has_illustration2 = illustration2_img_path is not None

    if not has_region and not has_illustration2:
        # 没有任何图片，显示提示
        error_text = "未找到地区相关图片和插图2，请确保同级目录存在符合要求的图片文件"
        current_y = draw_wrapped_text(
            draw, error_text, MARGIN + 20, current_y,
                              IMAGE_WIDTH - 2 * MARGIN - 40, FONT_CONTENT, (255, 0, 0)
        )
        block_end_y = current_y + BORDER_PADDING
        draw.rectangle([
            MARGIN - BORDER_PADDING, block_start_y - BORDER_PADDING,
            IMAGE_WIDTH - MARGIN + BORDER_PADDING, block_end_y
        ], outline=BORDER_COLOR, width=5)
        return block_end_y

    # 计算可用宽度
    available_width = IMAGE_WIDTH - 2 * MARGIN
    images = []
    max_height = 0  # 记录图片的最大高度，用于对齐

    # 只有一张插图的情况：居中放大显示
    if not has_region and has_illustration2:
        try:
            illustration_img = Image.open(illustration2_img_path)
            img_w, img_h = illustration_img.size

            # 计算缩放比例
            max_width = int(available_width * SINGLE_IMG_MAX_WIDTH_RATIO)
            if img_w > max_width:
                scale = max_width / img_w
                img_w = int(img_w * scale)
                img_h = int(img_h * scale)

            # 高质量缩放
            try:
                illustration_img = illustration_img.resize((img_w, img_h), Image.Resampling.LANCZOS)
            except AttributeError:
                illustration_img = illustration_img.resize((img_w, img_h), Image.LANCZOS)

            # 居中显示
            x_pos = (IMAGE_WIDTH - img_w) // 2
            images.append({
                'img': illustration_img,
                'width': img_w,
                'height': img_h,
                'x': x_pos,
                'y': current_y  # 顶部对齐
            })
            max_height = img_h
            print(f"   - 单张插图2（居中放大）：宽{img_w}px × 高{img_h}px")
        except Exception as e:
            error_text = f"插图2处理失败：{str(e)}"
            current_y = draw_wrapped_text(
                draw, error_text, MARGIN + 20, current_y,
                available_width, FONT_CONTENT, (255, 0, 0)
            )
    else:
        # 两张图片或只有地区图片的情况：左右并排
        single_img_max_width = (available_width - IMAGE_GAP) // 2  # 两张图片平分宽度（减去间距）

        # 处理地区图片
        if has_region:
            try:
                region_img = Image.open(region_img_path)
                img_w, img_h = region_img.size

                # 计算缩放比例
                if img_w > single_img_max_width:
                    scale = single_img_max_width / img_w
                    img_w = int(img_w * scale)
                    img_h = int(img_h * scale)

                # 高质量缩放
                try:
                    region_img = region_img.resize((img_w, img_h), Image.Resampling.LANCZOS)
                except AttributeError:
                    region_img = region_img.resize((img_w, img_h), Image.LANCZOS)

                images.append({
                    'img': region_img,
                    'width': img_w,
                    'height': img_h,
                    'x': MARGIN
                })
                max_height = max(max_height, img_h)
                print(f"   - 地区图片尺寸：宽{img_w}px × 高{img_h}px")
            except Exception as e:
                error_text = f"地区图片处理失败：{str(e)}"
                current_y = draw_wrapped_text(
                    draw, error_text, MARGIN + 20, current_y,
                    single_img_max_width, FONT_CONTENT, (255, 0, 0)
                )

        # 处理插图2（固定）
        if has_illustration2:
            try:
                illustration2_img = Image.open(illustration2_img_path)
                img_w, img_h = illustration2_img.size

                # 计算缩放比例
                if img_w > single_img_max_width:
                    scale = single_img_max_width / img_w
                    img_w = int(img_w * scale)
                    img_h = int(img_h * scale)

                # 高质量缩放
                try:
                    illustration2_img = illustration2_img.resize((img_w, img_h), Image.Resampling.LANCZOS)
                except AttributeError:
                    illustration2_img = illustration2_img.resize((img_w, img_h), Image.LANCZOS)

                # 计算x坐标（左侧图片右边 + 间距）
                x_pos = MARGIN + (single_img_max_width if has_region else 0) + IMAGE_GAP
                images.append({
                    'img': illustration2_img,
                    'width': img_w,
                    'height': img_h,
                    'x': x_pos
                })
                max_height = max(max_height, img_h)
                print(f"   - 插图2尺寸：宽{img_w}px × 高{img_h}px")
            except Exception as e:
                error_text = f"插图2处理失败：{str(e)}"
                x_pos = MARGIN + (single_img_max_width if has_region else 0) + IMAGE_GAP
                current_y = draw_wrapped_text(
                    draw, error_text, x_pos + 20, current_y,
                    single_img_max_width, FONT_CONTENT, (255, 0, 0)
                )

    # 绘制图片（垂直居中对齐）
    for img_data in images:
        # 计算y坐标使其垂直居中
        y_pos = current_y + (max_height - img_data['height']) // 2
        img.paste(img_data['img'], (img_data['x'], y_pos))

    # 更新y坐标
    current_y += max_height + BORDER_PADDING
    print(f"   - 图片板块结束y坐标：{current_y}px")

    # 绘制边框
    block_end_y = current_y + BORDER_PADDING
    draw.rectangle([
        MARGIN - BORDER_PADDING, block_start_y - BORDER_PADDING,
        IMAGE_WIDTH - MARGIN + BORDER_PADDING, block_end_y
    ], outline=BORDER_COLOR, width=5)

    return block_end_y


# -------------------------- 5. Excel处理函数 --------------------------
def get_excel_files():
    """查找同级目录的Excel文件"""
    excel_files = []
    for ext in ['*.xlsx', '*.xls']:
        for file in glob.glob(os.path.join(CURRENT_DIR, ext)):
            if os.path.dirname(os.path.abspath(file)) == CURRENT_DIR:
                excel_files.append(file)
    return excel_files


def extract_student_info(excel_path):
    """从Excel提取学生信息，新增弃考判断逻辑"""
    required_cols = ["姓名", "年龄", "年级", "地区", "YCL成绩", "蓝桥Stema成绩"]
    try:
        df = pd.read_excel(excel_path)
        missing_cols = [col for col in required_cols if col not in df.columns]
        if missing_cols:
            print(f"❌ Excel文件{os.path.basename(excel_path)}缺少必要列：{', '.join(missing_cols)}")
            return []

        students = []
        for _, row in df.iterrows():
            if pd.notna(row["姓名"]):
                # 处理年龄
                age_str = "未知"
                if pd.notna(row["年龄"]):
                    try:
                        age_int = int(float(str(row["年龄"]).strip()))
                        age_str = str(age_int)
                    except:
                        age_str = str(row["年龄"]).strip()

                # 处理YCL成绩（新增弃考判断）
                ycl_score = str(row["YCL成绩"]).strip() if pd.notna(row["YCL成绩"]) else "无"
                # 失败关键词包含弃考
                failed_keywords = ["未通过", "未过", "不通过", "未达标", "不合格", "弃考"]
                is_failed = any(keyword in ycl_score for keyword in failed_keywords)
                if is_failed:
                    ycl_score_clean = "无"  # 弃考或未通过均不显示证书
                else:
                    ycl_score_clean = ycl_score.replace("通过", "").replace("已过", "").strip()
                    if not ycl_score_clean and not is_failed:
                        ycl_score_clean = ycl_score.strip()

                # 处理蓝桥Stema成绩（新增弃考判断）
                lanqiao_raw = str(row["蓝桥Stema成绩"]).strip() if pd.notna(row["蓝桥Stema成绩"]) else "无"
                if "弃考" in lanqiao_raw:  # 包含弃考则不显示
                    has_lanqiao = "无"
                    lanqiao_score = "无"
                else:
                    has_lanqiao = "有" if (lanqiao_raw != "无") else "无"
                    lanqiao_score = lanqiao_raw

                student = {
                    "姓名": str(row["姓名"]).strip(),
                    "年龄": age_str,
                    "年级": str(row["年级"]).strip() if pd.notna(row["年级"]) else "未知",
                    "地区": str(row["地区"]).strip() if pd.notna(row["地区"]) else "未知",
                    "ycl_score": ycl_score_clean,
                    "has_lanqiao": has_lanqiao,
                    "lanqiao_score": lanqiao_score
                }
                if not any(s["姓名"] == student["姓名"] for s in students):
                    students.append(student)
        return students
    except Exception as e:
        print(f"❌ 读取Excel文件出错：{str(e)}")
        return []


# -------------------------- 6. 报告内容生成 --------------------------
def get_region_tier_prefixes(tier_regions):
    """提取梯队地区的前两字，返回集合"""
    return {get_region_prefix(region) for region in tier_regions}


def get_region_analysis(region):
    """生成地区分析"""
    region_prefix = get_region_prefix(region)
    print(f"   - 地区：{region}，提取前两字：{region_prefix}")

    tier1_prefixes = get_region_tier_prefixes(TIER1_REGIONS)
    tier2_prefixes = get_region_tier_prefixes(TIER2_REGIONS)

    if region_prefix in tier1_prefixes:
        return [
            "评级：★★★★★",
            "优势：拥有深厚的竞赛资源与机会倾斜，地区信息学普及高，升学政策较多，地区能长期稳定产出高排名选手，编程学习资历的认可度很高",
            "劣势：竞赛奖项和晋级名额有限，地区竞争压力较大，需要更早积累竞赛相关经验以脱颖而出"
        ]
    elif region_prefix in tier2_prefixes:
        return [
            "评级：★★★★",
            "优势：信息学发展规模逐渐扩大，家长意识逐渐增强，相较于第一梯队地区竞赛竞争压力相对较小，获奖分数线相对较低，学生更容易获得奖项！需保持学习优势，抢占升学先机",
            "劣势：当地专业师资团队较少，总体学员竞争奖项的激烈程度也在陆续加剧，需要尽快强化竞赛水平"
        ]
    else:
        return [
            "评级：★★★",
            "优势：编程普及度较低，参与信息学竞赛的人数较少竞争压力相对较小。在相同的获奖比例下，学生更容易获得奖项，早期建立编程学习，可快速取得升学优势",
            "劣势：优秀教学资源等方面明显不足，导致有编程学习潜力的学生因缺乏关键信息而滞后学习，需要通过线上渠道获取学习资料和竞赛学习规划"
        ]


def get_personal_report(student, end_month_str):
    """生成个性化报告内容"""
    region_prefix = get_region_prefix(student["地区"])
    ycl_score = student.get("ycl_score", "无")
    has_lanqiao = student.get("has_lanqiao", "无")
    lanqiao_score = student.get("lanqiao_score", "无")

    # 基础信息
    basic_info = [
        f"学生姓名：{student['姓名']}",
        f"所在年级：{student['年级']}",
        f"所在地区：{student['地区']}"
    ]
    if ycl_score != "无":
        basic_info.append(f"YCL证书：{ycl_score}")
    if has_lanqiao == "有":
        basic_info.append(f"蓝桥Stema参与情况：{lanqiao_score}")
    basic_info.append("当前阶段：图形化下册-多维算法")

    # 核心优势
    certificates = []
    if ycl_score != "无":
        certificates.append(f"YCL{ycl_score}")
    if has_lanqiao == "有":
        certificates.append(f"蓝桥Stema{lanqiao_score}")

    core_advantage_1 = "1. 掌握图形化编程知识(变量,逻辑判断,自定义,二分法等,具备项目思维)"
    if certificates:
        cert_str = "、".join(certificates)
        core_advantage_1 += f"，且拥有{cert_str}证书，获得至少一次的大型线上考试经验"

    core_advantage_2 = (
        "2. 学科学习压力小，编程学习时间充足，可进阶Python阶段，在大脑发育黄金期强化逻辑思维，掌握与计算机直接对话的语言技能")

    student_analysis = [
        "优势评级：★★★★",
        f"核心优势：",
        core_advantage_1,
        core_advantage_2
    ]

    # 年级定位分析
    grade_analysis = [
        "评级:★★★★",
        "核心定位:思维习惯培养黄金期",
        "1.可通过编程学习改善专注力差，粗心，逻辑不清等学习习惯，并辅助校内语数外主科的学习",
        "   语文：提升阅读理解,锻炼写作与表达，积累跨学科知识素材",
        "   数学：用加减乘除运算、角度、几何等多类型数学知识，强化孩子逻辑思维，抽象思维，提高解题能力",
        "   英语：(Python/C++) 寓教于乐，通过高频次练习积累词汇加强记忆，并提升孩子语感",
        "2.学习时间充足,可规划多次赛考,夯实基础后冲刺科技特长生认定,斩获信奥赛等高含金量奖项"
    ]

    # 阶段目标
    stage_goals = [
        f"1. 按计划{end_month_str}月份结束图形化阶段学习，需保质保量完成剩余课程，准备升阶段；",
        "2. 按计划Python1阶段学习四个月后，可准备必考的YCL四级考试，以考促学，增加学习自信；",
        "3. python一共三个阶段，会结合学习进度和比赛举办时间，就近规划国家级赛事，以赛代练，验收效果的同时提升学员竞赛心态及自身竞争力"
    ]

    # 阶段注意事项
    stage_note = "注意：Python每个阶段都有C++招募考试，若有明确科特规划且通过测试，可提前跳级进入C++学习，把握好“分岔口”，动态调整后续学习安排！"

    return {
        "main_title": f"{student['姓名']}同学专属编程成长计划",
        "sub_title": "（进则科技特长,退则数理化强）",
        "basic_info": basic_info,
        "grade_analysis": grade_analysis,
        "region_analysis": get_region_analysis(student["地区"]),
        "student_analysis": student_analysis,
        "stage_goals": stage_goals,
        "stage_note": stage_note,
    }


# -------------------------- 7. 生成报告图片 --------------------------
def create_report_image(student, end_month_str):
    """生成动态高度的报告图片，第五部分新增"插图"与插图1横向并排"""
    save_dir = os.path.join(CURRENT_DIR, "报告输出")
    if not os.path.exists(save_dir):
        os.makedirs(save_dir)

    report = get_personal_report(student, end_month_str)
    content_max_width = IMAGE_WIDTH - 2 * MARGIN
    available_width = content_max_width  # 可用宽度

    # 预计算所有板块的总高度
    print(f"\n📏 预计算报告各板块高度...")
    current_y = MARGIN  # 从顶部边距开始
    images_status = has_matched_images(student["地区"])
    print(f"   - 地区图片：{'存在' if images_status['has_region_img'] else '不存在'}")
    print(f"   - 插图：{'存在' if images_status['has_illustration_img'] else '不存在'}")
    print(f"   - 插图1：{'存在' if images_status['has_illustration1_img'] else '不存在'}")
    print(f"   - 插图2：{'存在' if images_status['has_illustration2_img'] else '不存在'}")

    # 1. 标题栏高度
    title_height = 220
    current_y += title_height + BLOCK_SPACING
    print(f"   1. 标题栏高度：{title_height}px → 当前累积y：{current_y}px")

    # 辅助函数：计算板块高度
    def calc_block_height(title, content, is_special=False):
        nonlocal current_y
        current_y += FONT_HEADER.size + HEADER_SPACING  # 标题高度
        content_width = content_max_width - 60 if is_special else content_max_width - 40
        temp_y = current_y

        # 计算内容高度
        for line in content:
            lines = []
            current_line = ""
            temp_draw = ImageDraw.Draw(Image.new('RGB', (1, 1)))
            for char in line:
                test_line = current_line + char
                if temp_draw.textlength(test_line, font=FONT_CONTENT) > content_width and current_line:
                    lines.append(current_line)
                    current_line = char
                else:
                    current_line = test_line
            if current_line:
                lines.append(current_line)
            temp_y += len(lines) * (FONT_CONTENT.size + LINE_SPACING)

        current_y = temp_y + BORDER_PADDING + BLOCK_SPACING  # 边框+间距
        return current_y

    # 计算板块1-4高度
    current_y = calc_block_height("一、学生基础信息", report["basic_info"])
    print(f"   2. 板块1后累积y：{current_y}px")
    current_y = calc_block_height("二、年级定位分析", report["grade_analysis"])
    print(f"   3. 板块2后累积y：{current_y}px")
    current_y = calc_block_height("三、地区梯队分析", report["region_analysis"], is_special=True)
    print(f"   4. 板块3后累积y：{current_y}px")
    current_y = calc_block_height("四、学生优势分析", report["student_analysis"])
    print(f"   5. 板块4后累积y：{current_y}px")

    # 计算板块5（新阶段规划）高度（包含插图1、插图和文字）
    # 获取两张图片：插图1和插图（横向并排）
    illustration1_path = match_illustration1_image()
    illustration_path = match_illustration_image()
    has_illustration1 = illustration1_path is not None
    has_illustration = illustration_path is not None

    # 计算图片总高度（取两张图的最大高度）
    max_img_height = 0
    if has_illustration1 or has_illustration:
        # 计算单张图片最大宽度（两张并排）
        single_img_max_width = int(available_width * DOUBLE_IMG_MAX_WIDTH_RATIO)

        # 计算插图1高度
        if has_illustration1:
            try:
                with Image.open(illustration1_path) as img:
                    img_w, img_h = img.size
                    if img_w > single_img_max_width:
                        scale = single_img_max_width / img_w
                        img_h = int(img_h * scale)
                    max_img_height = max(max_img_height, img_h)
            except:
                pass

        # 计算"插图"高度
        if has_illustration:
            try:
                with Image.open(illustration_path) as img:
                    img_w, img_h = img.size
                    if img_w > single_img_max_width:
                        scale = single_img_max_width / img_w
                        img_h = int(img_h * scale)
                    max_img_height = max(max_img_height, img_h)
            except:
                pass

    # 计算文字高度
    recommend_text = f"推荐{student['姓名']}同学在图形化下册课程结业后，迈入【Python】语言的进阶学习！"
    stage_goals = report["stage_goals"]
    stage_note = report["stage_note"]

    text_height = 0
    temp_draw = ImageDraw.Draw(Image.new('RGB', (1, 1)))

    # 推荐语高度
    lines = []
    current_line = ""
    for char in recommend_text:
        test_line = current_line + char
        if temp_draw.textlength(test_line, font=FONT_CONTENT) > (content_max_width - 40) and current_line:
            lines.append(current_line)
            current_line = char
        else:
            current_line = test_line
    if current_line:
        lines.append(current_line)
    text_height += len(lines) * (FONT_CONTENT.size + LINE_SPACING)

    # 阶段目标高度
    for line in stage_goals:
        lines = []
        current_line = ""
        for char in line:
            test_line = current_line + char
            if temp_draw.textlength(test_line, font=FONT_CONTENT) > (content_max_width - 60) and current_line:
                lines.append(current_line)
                current_line = char
            else:
                current_line = test_line
        if current_line:
            lines.append(current_line)
        text_height += len(lines) * (FONT_CONTENT.size + LINE_SPACING)

    # 注意事项高度
    lines = []
    current_line = ""
    for char in stage_note:
        test_line = current_line + char
        if temp_draw.textlength(test_line, font=FONT_CONTENT) > (content_max_width - 60) and current_line:
            lines.append(current_line)
            current_line = char
        else:
            current_line = test_line
    if current_line:
        lines.append(current_line)
    text_height += len(lines) * (FONT_CONTENT.size + LINE_SPACING)

    # 总高度 = 标题高度 + 图片高度 + 文字高度 + 间距
    current_y += FONT_HEADER.size + HEADER_SPACING  # 标题高度
    current_y += max_img_height + IMAGE_GAP if (has_illustration1 or has_illustration) else 0  # 图片高度+间距
    current_y += text_height  # 文字高度
    current_y += BORDER_PADDING + BLOCK_SPACING  # 边框内边距和板块间距
    print(f"   6. 板块5后累积y：{current_y}px")

    # 计算第六板块高度
    img_block_end_y = current_y
    if images_status['has_region_img'] or images_status['has_illustration2_img']:
        temp_img = Image.new("RGB", (IMAGE_WIDTH, 10000), BACKGROUND_COLOR)
        temp_draw = ImageDraw.Draw(temp_img)
        img_block_end_y = draw_images_section(temp_img, temp_draw, current_y, student["地区"])
        print(f"   7. 板块6后累积y：{img_block_end_y}px")

    # 最终报告高度
    final_image_height = img_block_end_y + MARGIN
    print(f"   8. 最终报告高度：{final_image_height}px")

    # 创建最终报告图片
    final_img = Image.new("RGB", (IMAGE_WIDTH, final_image_height), BACKGROUND_COLOR)
    final_draw = ImageDraw.Draw(final_img)

    # 绘制标题栏
    final_draw.rectangle([0, MARGIN, IMAGE_WIDTH, MARGIN + title_height], fill=TITLE_BAR_COLOR)
    # 主标题（居中）
    main_title_width = final_draw.textlength(report["main_title"], font=FONT_TITLE)
    main_title_x = (IMAGE_WIDTH - main_title_width) // 2
    main_title_y = MARGIN + 30
    final_draw.text((main_title_x, main_title_y), report["main_title"], fill="white", font=FONT_TITLE)
    # 副标题（居中）
    sub_title_width = final_draw.textlength(report["sub_title"], font=FONT_SUBTITLE)
    sub_title_x = (IMAGE_WIDTH - sub_title_width) // 2
    sub_title_y = main_title_y + FONT_TITLE.size + TITLE_LINE_SPACING
    final_draw.text((sub_title_x, sub_title_y), report["sub_title"], fill="white", font=FONT_SUBTITLE)

    # 绘制板块1-4
    current_y = MARGIN + title_height + BLOCK_SPACING

    def draw_block(title, content, is_special=False):
        nonlocal current_y
        block_start = current_y
        final_draw.text((MARGIN, current_y), title, fill=TEXT_COLOR_SUB, font=FONT_HEADER)
        current_y += FONT_HEADER.size + HEADER_SPACING

        content_width = content_max_width - 60 if is_special else content_max_width - 40
        for line in content:
            current_y = draw_wrapped_text(
                final_draw, line, MARGIN + 20, current_y, content_width, FONT_CONTENT, TEXT_COLOR_MAIN
            )

        block_end = current_y + BORDER_PADDING
        final_draw.rectangle([
            MARGIN - BORDER_PADDING, block_start - BORDER_PADDING,
            IMAGE_WIDTH - MARGIN + BORDER_PADDING, block_end
        ], outline=BORDER_COLOR, width=5)
        current_y = block_end + BLOCK_SPACING

    draw_block("一、学生基础信息", report["basic_info"])
    draw_block("二、年级定位分析", report["grade_analysis"])
    draw_block("三、地区梯队分析", report["region_analysis"], is_special=True)
    draw_block("四、学生优势分析", report["student_analysis"])

    # 绘制板块5（新阶段规划）- 核心修改部分
    block_start = current_y
    final_draw.text((MARGIN, current_y), "五、学习阶段规划", fill=TEXT_COLOR_SUB, font=FONT_HEADER)
    current_y += FONT_HEADER.size + HEADER_SPACING

    # 绘制插图1和"插图"横向并排（居中）
    illustration1_path = match_illustration1_image()
    illustration_path = match_illustration_image()
    has_illustration1 = illustration1_path is not None
    has_illustration = illustration_path is not None
    images = []
    max_height = 0

    if has_illustration1 or has_illustration:
        # 计算单张图片最大宽度（两张并排）
        single_img_max_width = int(available_width * DOUBLE_IMG_MAX_WIDTH_RATIO)

        # 处理插图1
        if has_illustration1:
            try:
                img = Image.open(illustration1_path)
                img_w, img_h = img.size
                # 缩放
                if img_w > single_img_max_width:
                    scale = single_img_max_width / img_w
                    img_w = int(img_w * scale)
                    img_h = int(img_h * scale)
                try:
                    img = img.resize((img_w, img_h), Image.Resampling.LANCZOS)
                except AttributeError:
                    img = img.resize((img_w, img_h), Image.LANCZOS)
                images.append({"img": img, "w": img_w, "h": img_h})
                max_height = max(max_height, img_h)
                print(f"   - 插图1尺寸：宽{img_w}px × 高{img_h}px")
            except Exception as e:
                error_text = f"插图1处理失败：{str(e)}"
                current_y = draw_wrapped_text(
                    final_draw, error_text, MARGIN + 20, current_y,
                    single_img_max_width, FONT_CONTENT, (255, 0, 0)
                )

        # 处理"插图"（放在插图1右侧）
        if has_illustration:
            try:
                img = Image.open(illustration_path)
                img_w, img_h = img.size
                # 缩放
                if img_w > single_img_max_width:
                    scale = single_img_max_width / img_w
                    img_w = int(img_w * scale)
                    img_h = int(img_h * scale)
                try:
                    img = img.resize((img_w, img_h), Image.Resampling.LANCZOS)
                except AttributeError:
                    img = img.resize((img_w, img_h), Image.LANCZOS)
                images.append({"img": img, "w": img_w, "h": img_h})
                max_height = max(max_height, img_h)
                print(f"   - 插图尺寸：宽{img_w}px × 高{img_h}px")
            except Exception as e:
                error_text = f"插图处理失败：{str(e)}"
                x_pos = MARGIN + single_img_max_width + IMAGE_GAP if has_illustration1 else MARGIN
                current_y = draw_wrapped_text(
                    final_draw, error_text, x_pos + 20, current_y,
                    single_img_max_width, FONT_CONTENT, (255, 0, 0)
                )

        # 计算总宽度和起始x坐标（整体居中）
        total_img_width = sum(img["w"] for img in images) + (IMAGE_GAP * (len(images) - 1))
        start_x = (IMAGE_WIDTH - total_img_width) // 2  # 整体居中
        current_x = start_x

        # 粘贴图片（垂直居中对齐）
        for img_data in images:
            y_pos = current_y + (max_height - img_data["h"]) // 2  # 垂直居中
            final_img.paste(img_data["img"], (current_x, y_pos))
            current_x += img_data["w"] + IMAGE_GAP  # 右侧留出间距

        current_y += max_height + IMAGE_GAP  # 图片高度 + 与文字的间距

    # 绘制文字内容
    recommend_text = f"推荐{student['姓名']}同学在图形化下册课程结业后，迈入【Python】语言的进阶学习！"
    current_y = draw_wrapped_text(
        final_draw, recommend_text, MARGIN + 40, current_y,
                                    content_max_width - 80, FONT_CONTENT, TEXT_COLOR_MAIN
    )
    current_y += LINE_SPACING

    # 阶段目标
    for line in report["stage_goals"]:
        current_y = draw_wrapped_text(
            final_draw, line, MARGIN + 40, current_y,
                              content_max_width - 80, FONT_CONTENT, TEXT_COLOR_MAIN
        )

    # 注意事项
    current_y += LINE_SPACING
    current_y = draw_wrapped_text(
        final_draw, report["stage_note"], MARGIN + 40, current_y,
                                          content_max_width - 80, FONT_CONTENT, TEXT_COLOR_MAIN
    )

    # 绘制边框
    block_end = current_y + BORDER_PADDING
    final_draw.rectangle(
        [MARGIN - BORDER_PADDING, block_start - BORDER_PADDING, IMAGE_WIDTH - MARGIN + BORDER_PADDING, block_end],
        outline=BORDER_COLOR, width=5)
    current_y = block_end + BLOCK_SPACING

    # 绘制第六板块
    if images_status['has_region_img'] or images_status['has_illustration2_img']:
        current_y = draw_images_section(final_img, final_draw, current_y, student["地区"])

    # 保存报告
    save_path = os.path.join(save_dir, f"{student['姓名']}_科技特长生成长计划.png")
    final_img.save(save_path, "PNG", quality=100, optimize=False)
    print(f"\n✅ 报告生成完成：{save_path}")
    return save_path


# -------------------------- 8. 批量生成报告 --------------------------
def batch_create_reports():
    """批量生成报告"""
    # 获取结束月份
    while True:
        try:
            end_month = input("请输入结束图形化阶段学习的月份（1-12）：")
            end_month = int(end_month)
            if 1 <= end_month <= 12:
                end_month_str = str(end_month)
                break
            else:
                print("请输入1-12之间的数字")
        except ValueError:
            print("请输入有效的数字（1-12）")

    excel_files = get_excel_files()
    if not excel_files:
        print(f"❌ 未找到Excel文件")
        return

    print(f"✅ 找到{len(excel_files)}个Excel文件：")
    for i, file in enumerate(excel_files, 1):
        print(f"   {i}. {os.path.basename(file)}")
    selected_idx = 0
    if len(excel_files) > 1:
        try:
            selected_idx = int(input(f"请选择文件（1-{len(excel_files)}）：")) - 1
            if not 0 <= selected_idx < len(excel_files):
                selected_idx = 0
                print("输入无效，默认选择第一个")
        except:
            selected_idx = 0
            print("输入无效，默认选择第一个")
    selected_excel = excel_files[selected_idx]

    students = extract_student_info(selected_excel)
    if not students:
        print(f"❌ 未提取到有效学生信息")
        return

    print(f"\n✅ 共提取到{len(students)}名学生，开始生成报告：")
    for i, student in enumerate(students, 1):
        print(f"\n=== 生成第{i}/{len(students)}个报告（{student['姓名']}）===")
        create_report_image(student, end_month_str)

    print(f"\n🎉 所有报告已保存至：{os.path.join(CURRENT_DIR, '报告输出')}")


# -------------------------- 9. 运行程序 --------------------------
if __name__ == "__main__":
    try:
        import pandas
        from PIL import Image
    except ImportError:
        print("请先安装依赖库：pip install pandas openpyxl pillow")
        exit(1)

    batch_create_reports()
