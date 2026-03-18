# GoHome 用户体验增强实现计划

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 实现 GoHome 项目的用户体验增强功能，包括 AI 家人问候、勋章系统、温馨治愈风格视觉优化、Vue 3 前端重构

**Architecture:**
- 保持现有 Flask 蓝图文体结构作为 API 后端
- 新增勋章模型层 (`models/badge.py`) 和 AI 问候模型层 (`models/ai_quote.py`)
- 新增勋章路由 (`routes/badges.py`) 和 API 路由 (`routes/api.py`)
- 数据存储在现有 JSON 文件基础上扩展 (`badges.json`, `ai_quotes.json`)
- 配置信息存储在 `config.py` 统一管理
- 新增 Vue 3 前端目录 (`frontend/`)，使用 Vite 构建

**Tech Stack:**
- 后端：Python 3.11, Flask, JSON 存储
- 前端：Vue 3.4+, Vite 5+, Pinia, Vue Router, Axios
- 样式：手写组件 + TailwindCSS (温馨治愈风格)

**设计文档:** `docs/superpowers/specs/2026-03-16-user-experience-enhancement-design.md`

**实施策略:**
1. 先实现后端数据模型和 API（Chunk 1-3）
2. 初始化 Vue 项目，配置开发环境（Chunk 4）
3. 实现 Vue 组件（签到页、勋章墙）（Chunk 5）
4. 集成测试与优化（Chunk 6）

---

## Chunk 1: 数据模型层

### Task 1: 创建勋章数据模型 (`models/badge.py`)

**Files:**
- Create: `models/badge.py`
- Reference: `models/storage.py`, `models/user.py`

- [ ] **Step 1: 创建勋章存储类**

```python
# models/badge.py

import json
import os
from datetime import datetime
from config import DATA_DIR


class BadgeStorage:
    """勋章数据存储"""

    BADGES_FILE = os.path.join(DATA_DIR, 'badges.json')

    # 勋章定义
    BADGE_DEFINITIONS = {
        # 时间里程碑系列
        'day_1': {'name': '🌱 初来乍到', 'description': '离开家乡的第一天', 'category': 'time'},
        'day_7': {'name': '📅 七日维新', 'description': '坚持了一周', 'category': 'time'},
        'day_30': {'name': '🌙 满月之约', 'description': '离家一个月了', 'category': 'time'},
        'day_100': {'name': '🔄 百日征程', 'description': '百日纪念', 'category': 'time'},
        'day_180': {'name': '💫 半年光阴', 'description': '半年过去了', 'category': 'time'},
        'day_365': {'name': '🏆 一年归期', 'description': '离家一整年', 'category': 'time'},

        # 签到连续系列
        'streak_3': {'name': '🔥 三日热', 'description': '开始上瘾了', 'category': 'streak'},
        'streak_7': {'name': '⚡ 周冠军', 'description': '坚持一周！', 'category': 'streak'},
        'streak_15': {'name': '🌟 半月谈', 'description': '半个月不间断', 'category': 'streak'},
        'streak_30': {'name': '💪 月全勤', 'description': '整月打卡', 'category': 'streak'},
        'streak_100': {'name': '👑 签到之王', 'description': '传奇！', 'category': 'streak'},

        # 特殊成就系列
        'late_night': {'name': '🌙 深夜未眠', 'description': '这么晚还没睡', 'category': 'special'},
        'early_bird': {'name': '🌅 早起鸟儿', 'description': '起得真早', 'category': 'special'},
        'quote_master': {'name': '📝 思乡达人', 'description': '文采斐然', 'category': 'special'},
    }

    @staticmethod
    def get_all():
        """获取所有用户的勋章数据"""
        try:
            if os.path.exists(BadgeStorage.BADGES_FILE):
                with open(BadgeStorage.BADGES_FILE, 'r', encoding='utf-8') as f:
                    return json.load(f)
        except (json.JSONDecodeError, IOError):
            pass
        return {}

    @staticmethod
    def save_all(badges):
        """保存勋章数据"""
        with open(BadgeStorage.BADGES_FILE, 'w', encoding='utf-8') as f:
            json.dump(badges, ensure_ascii=False, indent=2)

    @staticmethod
    def get_by_user(user_id):
        """获取用户的勋章列表"""
        badges = BadgeStorage.get_all()
        return badges.get(str(user_id), [])

    @staticmethod
    def add_badge(user_id, badge_id):
        """给用户添加勋章"""
        badges = BadgeStorage.get_all()
        user_badges = badges.get(str(user_id), [])

        # 检查是否已获得
        for badge in user_badges:
            if badge.get('badge_id') == badge_id:
                return None  # 已存在，不重复添加

        new_badge = {
            'badge_id': badge_id,
            'earned_at': datetime.now().isoformat()
        }
        user_badges.append(new_badge)
        badges[str(user_id)] = user_badges
        BadgeStorage.save_all(badges)
        return new_badge

    @staticmethod
    def get_definition(badge_id):
        """获取勋章定义"""
        return BadgeStorage.BADGE_DEFINITIONS.get(badge_id)

    @staticmethod
    def get_all_definitions():
        """获取所有勋章定义"""
        return BadgeStorage.BADGE_DEFINITIONS
```

- [ ] **Step 2: 运行语法检查**

```bash
cd /Users/old_people/test/GoHome
python -m py_compile models/badge.py
```
Expected: 无输出（无语法错误）

- [ ] **Step 3: 创建测试文件**

```python
# tests/test_badge.py

import pytest
import os
import json
from models.badge import BadgeStorage


class TestBadgeStorage:

    def setup_method(self):
        """测试前清理"""
        if os.path.exists(BadgeStorage.BADGES_FILE):
            os.remove(BadgeStorage.BADGES_FILE)

    def teardown_method(self):
        """测试后清理"""
        if os.path.exists(BadgeStorage.BADGES_FILE):
            os.remove(BadgeStorage.BADGES_FILE)

    def test_get_by_user_empty(self):
        """测试获取空用户勋章"""
        badges = BadgeStorage.get_by_user(1)
        assert badges == []

    def test_add_badge(self):
        """测试添加勋章"""
        result = BadgeStorage.add_badge(1, 'streak_3')
        assert result is not None
        assert result['badge_id'] == 'streak_3'

    def test_add_duplicate_badge(self):
        """测试重复添加勋章"""
        BadgeStorage.add_badge(1, 'streak_3')
        result = BadgeStorage.add_badge(1, 'streak_3')
        assert result is None  # 返回 None 表示已存在

    def test_get_definition(self):
        """测试获取勋章定义"""
        definition = BadgeStorage.get_definition('streak_3')
        assert definition['name'] == '🔥 三日热'
```

- [ ] **Step 4: 运行测试**

```bash
cd /Users/old_people/test/GoHome
pixi run pytest tests/test_badge.py -v
```
Expected: 4 个测试全部通过

- [ ] **Step 5: 提交**

```bash
cd /Users/old_people/test/GoHome
git add models/badge.py tests/test_badge.py
git commit -m "feat: add badge data model and storage"
```

---

### Task 2: 创建 AI 问候数据模型 (`models/ai_quote.py`)

**Files:**
- Create: `models/ai_quote.py`
- Reference: `models/storage.py`

- [ ] **Step 1: 创建 AI 问候存储类**

```python
# models/ai_quote.py

import json
import os
from datetime import datetime
from config import DATA_DIR


class AIQuoteStorage:
    """AI 问候数据存储"""

    QUOTES_FILE = os.path.join(DATA_DIR, 'ai_quotes.json')

    @staticmethod
    def get_all():
        """获取所有用户的 AI 问候数据"""
        try:
            if os.path.exists(AIQuoteStorage.QUOTES_FILE):
                with open(AIQuoteStorage.QUOTES_FILE, 'r', encoding='utf-8') as f:
                    return json.load(f)
        except (json.JSONDecodeError, IOError):
            pass
        return {}

    @staticmethod
    def save_all(quotes):
        """保存 AI 问候数据"""
        with open(AIQuoteStorage.QUOTES_FILE, 'w', encoding='utf-8') as f:
            json.dump(quotes, ensure_ascii=False, indent=2)

    @staticmethod
    def get_by_user(user_id, limit=None):
        """获取用户的 AI 问候列表"""
        quotes = AIQuoteStorage.get_all()
        user_quotes = quotes.get(str(user_id), [])

        # 按时间倒序排列
        user_quotes = sorted(
            user_quotes,
            key=lambda x: x.get('created_at', ''),
            reverse=True
        )

        if limit:
            user_quotes = user_quotes[:limit]

        return user_quotes

    @staticmethod
    def add_quote(user_id, content, family_role, dialect):
        """添加 AI 问候记录"""
        quotes = AIQuoteStorage.get_all()
        user_quotes = quotes.get(str(user_id), [])

        quote_id = max([q.get('id', 0) for q in user_quotes], default=0) + 1

        new_quote = {
            'id': quote_id,
            'user_id': user_id,
            'content': content,
            'family_role': family_role,
            'dialect': dialect,
            'created_at': datetime.now().isoformat()
        }
        user_quotes.append(new_quote)
        quotes[str(user_id)] = user_quotes
        AIQuoteStorage.save_all(quotes)
        return new_quote
```

- [ ] **Step 2: 运行语法检查**

```bash
cd /Users/old_people/test/GoHome
python -m py_compile models/ai_quote.py
```
Expected: 无输出

- [ ] **Step 3: 创建测试文件**

```python
# tests/test_ai_quote.py

import pytest
import os
from models.ai_quote import AIQuoteStorage


class TestAIQuoteStorage:

    def setup_method(self):
        if os.path.exists(AIQuoteStorage.QUOTES_FILE):
            os.remove(AIQuoteStorage.QUOTES_FILE)

    def teardown_method(self):
        if os.path.exists(AIQuoteStorage.QUOTES_FILE):
            os.remove(AIQuoteStorage.QUOTES_FILE)

    def test_get_by_user_empty(self):
        quotes = AIQuoteStorage.get_by_user(1)
        assert quotes == []

    def test_add_quote(self):
        result = AIQuoteStorage.add_quote(1, "娃啊，记得吃饭", "妈妈", "四川话")
        assert result['id'] == 1
        assert result['content'] == "娃啊，记得吃饭"

    def test_get_by_user_order(self):
        AIQuoteStorage.add_quote(1, "第一条", "妈妈", "四川话")
        import time
        time.sleep(0.01)
        AIQuoteStorage.add_quote(1, "第二条", "妈妈", "四川话")

        quotes = AIQuoteStorage.get_by_user(1)
        assert quotes[0]['content'] == "第二条"  # 最新在前
        assert quotes[1]['content'] == "第一条"
```

- [ ] **Step 4: 运行测试**

```bash
cd /Users/old_people/test/GoHome
pixi run pytest tests/test_ai_quote.py -v
```
Expected: 3 个测试全部通过

- [ ] **Step 5: 提交**

```bash
cd /Users/old_people/test/GoHome
git add models/ai_quote.py tests/test_ai_quote.py
git commit -m "feat: add AI quote data model and storage"
```

---

### Task 3: 扩展用户模型支持家人配置

**Files:**
- Modify: `models/user.py`
- Modify: `routes/auth.py`
- Reference: `config.py`

- [ ] **Step 1: 修改用户模型添加家人配置字段**

```python
# models/user.py - 修改 __init__ 方法

def __init__(self, user_data):
    self.id = user_data.get('id')
    self.username = user_data.get('username')
    self.password_hash = user_data.get('password_hash')
    self.hometown = user_data.get('hometown', '')
    self.current_city = user_data.get('current_city', '')
    self.leave_home_date = user_data.get('leave_home_date')
    self.created_at = user_data.get('created_at')
    # 新增家人配置字段
    self.family_role = user_data.get('family_role', '妈妈')
    self.nickname = user_data.get('nickname', '')
    self.tone_style = user_data.get('tone_style', '唠叨型')
```

- [ ] **Step 2: 修改注册表单添加家人配置**

```python
# routes/auth.py - 修改 register 函数

@auth_bp.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        username = request.form.get('username', '').strip()
        password = request.form.get('password', '')
        hometown = request.form.get('hometown', '').strip()
        current_city = request.form.get('current_city', '').strip()
        leave_home_date = request.form.get('leave_home_date', '')
        # 新增家人配置
        family_role = request.form.get('family_role', '妈妈')
        nickname = request.form.get('nickname', '').strip()
        tone_style = request.form.get('tone_style', '唠叨型')

        # ... 验证逻辑保持不变 ...

        # 创建用户
        user_data = {
            'username': username,
            'password_hash': generate_password_hash(password),
            'hometown': hometown,
            'current_city': current_city,
            'leave_home_date': leave_home_date,
            'created_at': datetime.now().isoformat(),
            'family_role': family_role,
            'nickname': nickname,
            'tone_style': tone_style,
        }
        created_user = UserStorage.create(user_data)
        # ... 后续保持不变 ...
```

- [ ] **Step 3: 修改注册模板添加家人配置表单**

在 `templates/register.html` 中添加：

```html
<!-- 在离家日期字段后添加 -->
<div class="form-group">
    <label for="family_role">家人称呼</label>
    <select id="family_role" name="family_role">
        <option value="妈妈">妈妈</option>
        <option value="爸爸">爸爸</option>
        <option value="奶奶">奶奶</option>
        <option value="姥姥">姥姥</option>
    </select>
</div>

<div class="form-group">
    <label for="nickname">对你的称呼</label>
    <input type="text" id="nickname" name="nickname" placeholder="如：娃、闺女">
</div>

<div class="form-group">
    <label for="tone_style">语气风格</label>
    <select id="tone_style" name="tone_style">
        <option value="唠叨型">唠叨型</option>
        <option value="含蓄型">含蓄型</option>
        <option value="直白型">直白型</option>
        <option value="幽默型">幽默型</option>
    </select>
</div>
```

- [ ] **Step 4: 运行测试验证注册流程**

```bash
cd /Users/old_people/test/GoHome
pixi run pytest tests/test_auth.py -v -k register
```
Expected: 现有注册测试通过（如有）

- [ ] **Step 5: 提交**

```bash
cd /Users/old_people/test/GoHome
git add models/user.py routes/auth.py templates/register.html
git commit -m "feat: add family role configuration to user registration"
```

---

## Chunk 2: 路由和 API 层

### Task 4: 创建勋章路由

**Files:**
- Create: `routes/badges.py`
- Modify: `app.py`
- Create: `templates/badges.html`

- [ ] **Step 1: 创建勋章路由文件**

```python
# routes/badges.py

from flask import Blueprint, render_template, session, jsonify
from routes.auth import login_required
from models.badge import BadgeStorage
from models.user import User
from models.storage import UserStorage

badges_bp = Blueprint('badges', __name__)


@badges_bp.route('/badges')
@login_required
def index():
    """勋章墙页面"""
    user = User(UserStorage.get_by_id(session['user_id']))
    user_badges = BadgeStorage.get_by_user(user.id)
    all_definitions = BadgeStorage.get_all_definitions()

    # 获取用户已获得的勋章 ID 列表
    earned_ids = set(b['badge_id'] for b in user_badges)

    # 按分类组织勋章
    categories = {
        'time': {'name': '时间里程碑', 'badges': []},
        'streak': {'name': '签到连续', 'badges': []},
        'special': {'name': '特殊成就', 'badges': []},
    }

    for badge_id, definition in all_definitions.items():
        category = definition.get('category', 'special')
        categories[category]['badges'].append({
            'id': badge_id,
            'name': definition['name'],
            'description': definition['description'],
            'earned': badge_id in earned_ids,
            'earned_at': next(
                (b['earned_at'] for b in user_badges if b['badge_id'] == badge_id),
                None
            )
        })

    return render_template('badges.html', user=user, categories=categories)


@badges_bp.route('/api/badges')
@login_required
def api_list():
    """API: 获取用户勋章列表"""
    user_badges = BadgeStorage.get_by_user(session['user_id'])
    all_definitions = BadgeStorage.get_all_definitions()

    result = []
    for badge in user_badges:
        definition = all_definitions.get(badge['badge_id'], {})
        result.append({
            'id': badge['badge_id'],
            'name': definition.get('name', ''),
            'description': definition.get('description', ''),
            'category': definition.get('category', ''),
            'earned_at': badge['earned_at']
        })

    return jsonify(result)
```

- [ ] **Step 2: 注册蓝图到 app.py**

```python
# app.py

from routes.badges import badges_bp

def create_app():
    app = Flask(__name__)
    app.secret_key = SECRET_KEY
    app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(days=7)

    # 注册蓝图
    app.register_blueprint(auth_bp)
    app.register_blueprint(checkin_bp)
    app.register_blueprint(dashboard_bp)
    app.register_blueprint(quotes_bp)
    app.register_blueprint(ai_hometown_bp)
    app.register_blueprint(badges_bp)  # 新增

    # ... 错误处理保持不变 ...

    return app
```

- [ ] **Step 3: 创建勋章页面模板**

```html
<!-- templates/badges.html -->
{% extends "base.html" %}

{% block title %}我的勋章{% endblock %}

{% block content %}
<div class="badges-page">
    <h1>我的勋章墙</h1>
    <p class="subtitle">记录你的每一个成长瞬间</p>

    {% for category_id, category in categories.items() %}
    <div class="badge-section">
        <h2>{{ category.name }}</h2>
        <div class="badge-grid">
            {% for badge in category.badges %}
            <div class="badge-card {% if not badge.earned %}locked{% endif %}"
                 title="{{ badge.description }}">
                <div class="badge-icon">
                    {{ badge.name.split(' ')[0] if ' ' in badge.name else '🔒' }}
                </div>
                <div class="badge-info">
                    <h3>{{ badge.name }}</h3>
                    <p class="badge-description">{{ badge.description }}</p>
                    {% if badge.earned %}
                    <p class="badge-earned">获得时间：{{ badge.earned_at[:10] }}</p>
                    {% else %}
                    <p class="badge-locked">🔒 未解锁</p>
                    {% endif %}
                </div>
            </div>
            {% endfor %}
        </div>
    </div>
    {% endfor %}
</div>

<style>
.badges-page {
    max-width: 960px;
    margin: 0 auto;
    padding: 20px;
}

.badge-section {
    margin-bottom: 40px;
}

.badge-section h2 {
    color: #333;
    margin-bottom: 20px;
}

.badge-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 20px;
}

.badge-card {
    background: #fff;
    border-radius: 12px;
    padding: 20px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.08);
    transition: transform 0.2s;
}

.badge-card:hover {
    transform: translateY(-4px);
}

.badge-card.locked {
    opacity: 0.6;
    filter: grayscale(80%);
}

.badge-icon {
    font-size: 48px;
    text-align: center;
    margin-bottom: 10px;
}

.badge-info h3 {
    font-size: 16px;
    margin-bottom: 8px;
}

.badge-description {
    font-size: 14px;
    color: #666;
    margin-bottom: 8px;
}

.badge-earned {
    font-size: 12px;
    color: #98D8AA;
}

.badge-locked {
    font-size: 12px;
    color: #888;
}
</style>
{% endblock %}
```

- [ ] **Step 4: 创建测试文件**

```python
# tests/test_badges_route.py

import pytest
from app import create_app


class TestBadgesRoute:

    def setup_method(self):
        self.app = create_app()
        self.app.config['TESTING'] = True
        self.client = self.app.test_client()

    def test_badges_page_requires_login(self):
        """测试勋章页需要登录"""
        response = self.client.get('/badges')
        assert response.status_code == 302  # 重定向到登录页

    def test_badges_page_accessible_after_login(self):
        """测试登录后访问勋章页"""
        with self.client as c:
            with c.session_transaction() as sess:
                sess['user_id'] = 1
            response = c.get('/badges')
            assert response.status_code == 200

    def test_api_badges_requires_login(self):
        """测试 API 需要登录"""
        response = self.client.get('/api/badges')
        assert response.status_code == 302
```

- [ ] **Step 5: 运行测试**

```bash
cd /Users/old_people/test/GoHome
pixi run pytest tests/test_badges_route.py -v
```
Expected: 3 个测试通过

- [ ] **Step 6: 提交**

```bash
cd /Users/old_people/test/GoHome
git add routes/badges.py templates/badges.html app.py tests/test_badges_route.py
git commit -m "feat: add badges route and page template"
```

---

## Chunk 3: 签到流程整合

### Task 5: 修改签到流程整合勋章检查和 AI 问候

**Files:**
- Modify: `routes/checkin.py`
- Reference: `models/badge.py`, `models/ai_quote.py`, `utils/ai_hometown_generator.py`

- [ ] **Step 1: 添加勋章检查逻辑到签到函数**

```python
# routes/checkin.py - 修改 do_checkin 函数

from models.badge import BadgeStorage
from models.ai_quote import AIQuoteStorage
from utils.ai_hometown_generator import AIHometownGenerator

@checkin_bp.route('/checkin', methods=['GET', 'POST'])
@login_required
def do_checkin():
    user = User(UserStorage.get_by_id(session['user_id']))
    new_badges = []  # 记录新解锁的勋章

    if request.method == 'POST':
        # 检查今天是否已签到
        if user.has_checked_in_today():
            flash('今天已经签到过了', 'info')
            return redirect(url_for('dashboard.index'))

        # 检查特殊成就勋章
        checkin_time = datetime.now()

        # 深夜未眠勋章 (凌晨 2 点后)
        if checkin_time.hour >= 2:
            badge = BadgeStorage.add_badge(user.id, 'late_night')
            if badge:
                new_badges.append(BadgeStorage.get_definition('late_night'))

        # 早起鸟儿勋章 (早上 6 点前)
        elif checkin_time.hour < 6:
            badge = BadgeStorage.add_badge(user.id, 'early_bird')
            if badge:
                new_badges.append(BadgeStorage.get_definition('early_bird'))

        # 优先尝试 AI 生成思乡话语（家人问候）
        quote_content = None
        if AI_API_KEY:
            try:
                generator = AIHometownGenerator(
                    api_key=AI_API_KEY,
                    base_url=AI_API_BASE_URL,
                    model=AI_MODEL
                )
                user_info = {
                    'hometown': user.hometown,
                    'current_city': getattr(user, 'current_city', ''),
                    'family_role': getattr(user, 'family_role', '妈妈'),
                    'nickname': getattr(user, 'nickname', '娃'),
                    'tone_style': getattr(user, 'tone_style', '唠叨型'),
                }
                ai_result = generator.generate_family_greeting(user_info)
                if ai_result:
                    quote_content = ai_result['content']
                    # 保存 AI 问候记录
                    AIQuoteStorage.add_quote(
                        user.id,
                        quote_content,
                        user_info['family_role'],
                        ai_result.get('dialect', '')
                    )
            except Exception as e:
                print(f"AI 生成失败：{e}")

        # 如果 AI 生成失败，使用内置话语
        if not quote_content:
            quote = QuoteGenerator.get_random_quote(user.id)
            quote_content = quote['content']

        # 创建签到记录
        checkin_data = {
            'checkin_date': datetime.now().strftime('%Y-%m-%d'),
            'checkin_time': datetime.now().isoformat(),
            'quote_content': quote_content
        }
        CheckinStorage.add_checkin(user.id, checkin_data)

        # 检查时间里程碑勋章
        days_away = user.get_days_away_from_home()
        time_badges = {
            1: 'day_1', 7: 'day_7', 30: 'day_30',
            100: 'day_100', 180: 'day_180', 365: 'day_365'
        }
        for threshold, badge_id in time_badges.items():
            if days_away == threshold:
                badge = BadgeStorage.add_badge(user.id, badge_id)
                if badge:
                    new_badges.append(BadgeStorage.get_definition(badge_id))

        # 检查连续签到勋章
        stats = user.get_checkin_stats()
        streak_badges = {
            3: 'streak_3', 7: 'streak_7', 15: 'streak_15',
            30: 'streak_30', 100: 'streak_100'
        }
        for threshold, badge_id in streak_badges.items():
            if stats['current_streak'] == threshold:
                badge = BadgeStorage.add_badge(user.id, badge_id)
                if badge:
                    new_badges.append(BadgeStorage.get_definition(badge_id))

        flash('签到成功！', 'success')
        return render_template(
            'checkin_result.html',
            user=user,
            quote={'content': quote_content, 'category': 'AI 生成' if AI_API_KEY else '内置'},
            new_badges=new_badges  # 传递新解锁的勋章
        )

    # GET 请求逻辑保持不变
    if user.has_checked_in_today():
        flash('今天已经签到过了', 'info')
        return redirect(url_for('dashboard.index'))

    return render_template('checkin.html', user=user)
```

- [ ] **Step 2: 修改签到结果模板显示家人问候和新勋章**

```html
<!-- templates/checkin_result.html - 修改内容 -->
<div class="checkin-result">
    <h1>签到成功！</h1>

    <!-- 家人问候区域 -->
    <div class="family-greeting">
        <div class="greeting-header">
            <span class="family-role">{{ user.family_role or '妈妈' }}</span>
            <span class="greeting-dialect">{{ quote.dialect or '' }}</span>
        </div>
        <p class="greeting-content">{{ quote.content }}</p>
    </div>

    <!-- 新解锁勋章区域 -->
    {% if new_badges %}
    <div class="new-badges">
        <h2>🎉 恭喜解锁新勋章!</h2>
        <div class="badge-showcase">
            {% for badge in new_badges %}
            <div class="badge-card animate-in">
                <div class="badge-icon">{{ badge.name.split(' ')[0] }}</div>
                <div class="badge-info">
                    <h3>{{ badge.name }}</h3>
                    <p>{{ badge.description }}</p>
                </div>
            </div>
            {% endfor %}
        </div>
    </div>
    {% endif %}

    <!-- 签到统计 -->
    <div class="stats-summary">
        <div class="stat-item">
            <span class="stat-value">{{ stats.current_streak }}</span>
            <span class="stat-label">连续签到天数</span>
        </div>
        <div class="stat-item">
            <span class="stat-value">{{ stats.total_days }}</span>
            <span class="stat-label">总签到天数</span>
        </div>
    </div>

    <a href="{{ url_for('dashboard.index') }}" class="btn-primary">返回首页</a>
</div>

<style>
.family-greeting {
    background: linear-gradient(135deg, #FFF5F5 0%, #FFF0E6 100%);
    border-radius: 16px;
    padding: 24px;
    margin: 20px 0;
    border: 2px solid #FFB6C1;
}

.greeting-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 12px;
}

.family-role {
    font-size: 18px;
    font-weight: bold;
    color: #F5A623;
}

.greeting-dialect {
    font-size: 14px;
    color: #888;
}

.greeting-content {
    font-size: 18px;
    line-height: 1.6;
    color: #333;
    font-style: italic;
}

.new-badges {
    background: linear-gradient(135deg, #FFF9E6 0%, #FFF0E6 100%);
    border-radius: 16px;
    padding: 24px;
    margin: 20px 0;
    border: 2px solid #F5A623;
}

.new-badges h2 {
    text-align: center;
    margin-bottom: 20px;
}

.badge-showcase {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 16px;
}

.badge-card {
    background: #fff;
    border-radius: 12px;
    padding: 16px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.12);
    animation: badge-pop 0.5s ease-out;
}

@keyframes badge-pop {
    0% { transform: scale(0); opacity: 0; }
    50% { transform: scale(1.1); }
    100% { transform: scale(1); opacity: 1; }
}

.animate-in {
    animation: badge-pop 0.5s ease-out;
}
</style>
```

- [ ] **Step 3: 修改 AI 生成器添加家人问候方法**

```python
# utils/ai_hometown_generator.py - 添加新方法

def generate_family_greeting(self, user_info: dict) -> Optional[dict]:
    """
    为签到打卡生成家人问候语

    Args:
        user_info: 用户信息，包含 hometown, family_role, nickname, tone_style

    Returns:
        生成结果
    """
    hometown = user_info.get('hometown', '')
    family_role = user_info.get('family_role', '妈妈')
    nickname = user_info.get('nickname', '娃')
    tone_style = user_info.get('tone_style', '唠叨型')

    if not hometown:
        return None

    dialect = self._get_dialect(hometown)

    # 根据语气风格构建提示词
    tone_prompts = {
        '唠叨型': '像爱唠叨的妈妈，事无巨细地关心',
        '含蓄型': '像不善言辞的父亲，关心藏在心里',
        '直白型': '像直爽的家人，有话直说',
        '幽默型': '像幽默的家人，用玩笑表达关心',
    }
    tone_desc = tone_prompts.get(tone_style, '温馨关怀的语气')

    prompt = f'''请扮演用户的{family_role}，用{dialect}方言风格，{tone_desc}，
给用户（你叫 TA{nickname}）写一句日常问候语。

要求：
- 口语化，像真正的家人对话
- 体现对游子生活的关心（吃饭、穿衣、休息）
- 20-50 字
- 不要有 AI 生成的痕迹，要像真人说话'''

    result = self._call_api(prompt)

    if result:
        result = result.strip().strip('"\'""')
        return {
            'content': result,
            'family_role': family_role,
            'dialect': dialect,
            'nickname': nickname
        }
    return None
```

- [ ] **Step 4: 运行测试**

```bash
cd /Users/old_people/test/GoHome
python -m py_compile routes/checkin.py
python -m py_compile utils/ai_hometown_generator.py
```
Expected: 无语法错误

- [ ] **Step 5: 提交**

```bash
cd /Users/old_people/test/GoHome
git add routes/checkin.py templates/checkin_result.html utils/ai_hometown_generator.py
git commit -m "feat: integrate badge check and AI family greeting into checkin flow"
```

---

## Chunk 4: 视觉风格优化

### Task 6: 更新 CSS 为温馨治愈风格

**Files:**
- Modify: `static/css/style.css`
- Reference: 设计文档中的色彩规范

- [ ] **Step 1: 更新全局样式变量**

```css
/* static/css/style.css */

:root {
    /* 温馨治愈风格配色 */
    --color-primary: #F5A623;
    --color-primary-light: #FFB347;
    --color-background: #FAF8F5;
    --color-card: #FFFFFF;
    --color-text: #333333;
    --color-text-muted: #888888;
    --color-accent: #FFB6C1;
    --color-success: #98D8AA;
    --color-error: #FF6B6B;

    /* 圆角 */
    --border-radius-sm: 8px;
    --border-radius-md: 12px;
    --border-radius-lg: 16px;

    /* 阴影 */
    --shadow-sm: 0 2px 4px rgba(0,0,0,0.06);
    --shadow-md: 0 4px 8px rgba(0,0,0,0.08);
    --shadow-lg: 0 8px 16px rgba(0,0,0,0.12);
}

body {
    background-color: var(--color-background);
    color: var(--color-text);
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

/* 卡片样式 */
.card {
    background: var(--color-card);
    border-radius: var(--border-radius-md);
    box-shadow: var(--shadow-md);
    padding: 20px;
}

/* 按钮样式 */
.btn {
    border-radius: var(--border-radius-sm);
    padding: 10px 20px;
    font-weight: 500;
    transition: all 0.2s;
}

.btn:hover {
    transform: translateY(-2px);
    box-shadow: var(--shadow-md);
}

.btn-primary {
    background: linear-gradient(135deg, var(--color-primary) 0%, var(--color-primary-light) 100%);
    border: none;
    color: white;
}

/* 输入框样式 */
.form-group input,
.form-group select,
.form-group textarea {
    border-radius: var(--border-radius-sm);
    border: 2px solid #E8E8E8;
    padding: 12px;
    transition: border-color 0.2s;
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
    outline: none;
    border-color: var(--color-primary);
}

/* 导航栏 */
.navbar {
    background: var(--color-card);
    box-shadow: var(--shadow-sm);
}
```

- [ ] **Step 2: 提交**

```bash
cd /Users/old_people/test/GoHome
git add static/css/style.css
git commit -m "style: update CSS to warm healing style theme"
```

---

### Task 7: 添加勋章图标资源

**Files:**
- Create: `static/images/badges/day_1.png` (及其他勋章图标)

- [ ] **Step 1: 创建简单的 emoji 占位图标**

由于无法直接生成 PNG 图标，我们先用 emoji 作为临时方案：

```python
# scripts/create_badge_placeholders.py

import os

BADGE_EMOJIS = {
    'day_1': '🌱',
    'day_7': '📅',
    'day_30': '🌙',
    'day_100': '🔄',
    'day_180': '💫',
    'day_365': '🏆',
    'streak_3': '🔥',
    'streak_7': '⚡',
    'streak_15': '🌟',
    'streak_30': '💪',
    'streak_100': '👑',
    'late_night': '🌙',
    'early_bird': '🌅',
    'quote_master': '📝',
}

# 创建目录
os.makedirs('static/images/badges', exist_ok=True)

# 创建 SVG 图标（简单方案）
for badge_id, emoji in BADGE_EMOJIS.items():
    svg_content = f'''<svg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 64 64">
  <rect width="64" height="64" rx="12" fill="#FFF5E6"/>
  <text x="32" y="42" font-size="32" text-anchor="middle">{emoji}</text>
</svg>'''

    with open(f'static/images/badges/{badge_id}.svg', 'w', encoding='utf-8') as f:
        f.write(svg_content)

print("Badge placeholders created!")
```

- [ ] **Step 2: 运行脚本创建图标**

```bash
cd /Users/old_people/test/GoHome
python scripts/create_badge_placeholders.py
```

- [ ] **Step 3: 提交**

```bash
cd /Users/old_people/test/GoHome
git add scripts/create_badge_placeholders.py static/images/badges/
git commit -m "feat: add badge icon placeholders (SVG)"
```

---

## Chunk 5: 完善与测试

### Task 8: 端到端测试

- [ ] **Step 1: 创建端到端测试文件**

```python
# tests/test_e2e_checkin.py

import pytest
from app import create_app
from models.storage import UserStorage, CheckinStorage
from models.badge import BadgeStorage
from datetime import datetime


class TestE2ECheckin:

    def setup_method(self):
        self.app = create_app()
        self.app.config['TESTING'] = True
        self.app.config['SECRET_KEY'] = 'test-secret'
        self.client = self.app.test_client()

        # 清理测试数据
        for f in ['badges.json', 'ai_quotes.json']:
            try:
                os.remove(f)
            except:
                pass

    def test_checkin_grants_streak_badge(self):
        """测试连续签到获得勋章"""
        with self.client as c:
            # 创建测试用户
            with c.session_transaction() as sess:
                sess['user_id'] = 1

            # 模拟连续 3 天签到
            for i in range(3):
                response = c.post('/checkin', data={'checkin': '1'})
                assert response.status_code == 200

            # 检查是否获得 streak_3 勋章
            badges = BadgeStorage.get_by_user(1)
            streak_badge = any(b['badge_id'] == 'streak_3' for b in badges)
            assert streak_badge
```

- [ ] **Step 2: 手动测试检查清单**

创建 `docs/manual-test-checklist.md`:

```markdown
# 手动测试检查清单

## 注册流程
- [ ] 填写用户名、密码、家乡、当前城市
- [ ] 选择家人称呼（妈妈/爸爸/奶奶等）
- [ ] 填写对你的称呼
- [ ] 选择语气风格
- [ ] 提交后成功跳转登录页

## 签到流程
- [ ] 点击签到按钮
- [ ] 签到成功后显示家人问候语
- [ ] 问候语有方言特色
- [ ] 如果解锁勋章，显示勋章弹窗

## 勋章系统
- [ ] 访问/勋章墙页面能看到所有勋章分类
- [ ] 已获得的勋章显示彩色
- [ ] 未获得的勋章显示灰色锁定状态
- [ ] 悬停勋章显示详情

## 视觉风格
- [ ] 整体暖色调
- [ ] 卡片圆角
- [ ] 按钮悬停有动画
- [ ] 移动端响应式正常
```

- [ ] **Step 3: 提交**

```bash
cd /Users/old_people/test/GoHome
git add tests/test_e2e_checkin.py docs/manual-test-checklist.md
git commit -m "test: add e2e tests and manual test checklist"
```

---

## Chunk 6: Vue 前端重构

### Task 9: 初始化 Vue 3 + Vite 项目

**Files:**
- Create: `frontend/package.json`
- Create: `frontend/vite.config.js`
- Create: `frontend/index.html`
- Create: `frontend/src/main.js`
- Create: `frontend/src/App.vue`

- [ ] **Step 1: 创建 package.json**

```json
{
  "name": "homesignin-frontend",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "vue": "^3.4.0",
    "vue-router": "^4.2.0",
    "pinia": "^2.1.0",
    "axios": "^1.6.0"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^5.0.0",
    "vite": "^5.0.0",
    "tailwindcss": "^3.4.0",
    "postcss": "^8.4.0",
    "autoprefixer": "^10.4.0"
  }
}
```

- [ ] **Step 2: 创建 vite.config.js**

```javascript
// frontend/vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:5001',
        changeOrigin: true
      },
      '/static': {
        target: 'http://localhost:5001',
        changeOrigin: true
      }
    }
  }
})
```

- [ ] **Step 3: 创建 index.html**

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8">
    <link rel="icon" href="/favicon.ico">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>思乡签到 - HomeSignin</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

- [ ] **Step 4: 创建 src/main.js**

```javascript
// frontend/src/main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import router from './router'
import './styles/main.css'

const app = createApp(App)
const pinia = createPinia()

app.use(pinia)
app.use(router)
app.mount('#app')
```

- [ ] **Step 5: 创建 src/App.vue**

```vue
<!-- frontend/src/App.vue -->
<template>
  <div id="app">
    <nav class="navbar" v-if="showNav">
      <div class="nav-container">
        <router-link to="/" class="logo">思乡签到</router-link>
        <div class="nav-links">
          <router-link to="/dashboard">首页</router-link>
          <router-link to="/checkin">签到</router-link>
          <router-link to="/badges">勋章</router-link>
          <router-link to="/profile">我的</router-link>
        </div>
      </div>
    </nav>
    <main class="main-content">
      <router-view />
    </main>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const showNav = computed(() => !['/login', '/register'].includes(route.path))
</script>

<style scoped>
#app {
  min-height: 100vh;
  background: #FAF8F5;
}

.navbar {
  background: #fff;
  box-shadow: 0 2px 4px rgba(0,0,0,0.06);
  padding: 12px 0;
}

.nav-container {
  max-width: 960px;
  margin: 0 auto;
  padding: 0 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.logo {
  font-size: 20px;
  font-weight: bold;
  color: #F5A623;
  text-decoration: none;
}

.nav-links {
  display: flex;
  gap: 24px;
}

.nav-links a {
  color: #333;
  text-decoration: none;
  transition: color 0.2s;
}

.nav-links a:hover {
  color: #F5A623;
}

.main-content {
  max-width: 960px;
  margin: 0 auto;
  padding: 24px 20px;
}
</style>
```

- [ ] **Step 6: 创建 src/styles/main.css**

```css
/* frontend/src/styles/main.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --color-primary: #F5A623;
  --color-primary-light: #FFB347;
  --color-background: #FAF8F5;
  --color-card: #FFFFFF;
  --color-text: #333333;
  --color-text-muted: #888888;
  --color-accent: #FFB6C1;
  --color-success: #98D8AA;
  --color-error: #FF6B6B;
  --border-radius-sm: 8px;
  --border-radius-md: 12px;
  --border-radius-lg: 16px;
}

body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  background: var(--color-background);
  color: var(--color-text);
}

/* 通用组件 */
.card {
  background: var(--color-card);
  border-radius: var(--border-radius-md);
  box-shadow: 0 4px 8px rgba(0,0,0,0.08);
  padding: 20px;
}

.btn {
  border-radius: var(--border-radius-sm);
  padding: 10px 20px;
  font-weight: 500;
  transition: all 0.2s;
  cursor: pointer;
  border: none;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.12);
}

.btn-primary {
  background: linear-gradient(135deg, var(--color-primary) 0%, var(--color-primary-light) 100%);
  color: white;
}
```

- [ ] **Step 7: 创建 tailwind.config.js**

```javascript
// frontend/tailwind.config.js
export default {
  content: [
    "./index.html",
    "./src/**/*.{vue,js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: '#F5A623',
        'primary-light': '#FFB347',
        background: '#FAF8F5',
        card: '#FFFFFF',
        accent: '#FFB6C1',
        success: '#98D8AA',
        error: '#FF6B6B',
      },
      borderRadius: {
        sm: '8px',
        md: '12px',
        lg: '16px',
      },
    },
  },
  plugins: [],
}
```

- [ ] **Step 8: 创建 postcss.config.js**

```javascript
// frontend/postcss.config.js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

- [ ] **Step 9: 安装依赖**

```bash
cd frontend
npm install
```
Expected: 安装成功，无错误

- [ ] **Step 10: 提交**

```bash
cd /Users/old_people/test/GoHome
git add frontend/
git commit -m "feat: initialize Vue 3 + Vite frontend project"
```

---

### Task 10: 创建 Vue Router 和 Pinia Store

**Files:**
- Create: `frontend/src/router/index.js`
- Create: `frontend/src/stores/user.js`
- Create: `frontend/src/stores/badges.js`
- Create: `frontend/src/stores/checkin.js`
- Create: `frontend/src/api/index.js`

- [ ] **Step 1: 创建路由配置**

```javascript
// frontend/src/router/index.js
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/',
    name: 'Home',
    redirect: '/dashboard'
  },
  {
    path: '/login',
    name: 'Login',
    component: () => import('../views/Login.vue')
  },
  {
    path: '/register',
    name: 'Register',
    component: () => import('../views/Register.vue')
  },
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: () => import('../views/Dashboard.vue'),
    meta: { requiresAuth: true }
  },
  {
    path: '/checkin',
    name: 'Checkin',
    component: () => import('../views/Checkin.vue'),
    meta: { requiresAuth: true }
  },
  {
    path: '/badges',
    name: 'Badges',
    component: () => import('../views/Badges.vue'),
    meta: { requiresAuth: true }
  },
  {
    path: '/profile',
    name: 'Profile',
    component: () => import('../views/Profile.vue'),
    meta: { requiresAuth: true }
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// 路由守卫
router.beforeEach((to, from, next) => {
  const isAuthenticated = localStorage.getItem('user_id')

  if (to.meta.requiresAuth && !isAuthenticated) {
    next('/login')
  } else if (to.path === '/login' && isAuthenticated) {
    next('/dashboard')
  } else {
    next()
  }
})

export default router
```

- [ ] **Step 2: 创建 API 封装**

```javascript
// frontend/src/api/index.js
import axios from 'axios'

const api = axios.create({
  baseURL: '/api',
  withCredentials: true
})

// 响应拦截器
api.interceptors.response.use(
  response => response.data,
  error => {
    if (error.response?.status === 401) {
      localStorage.removeItem('user_id')
      window.location.href = '/login'
    }
    return Promise.reject(error)
  }
)

export const authAPI = {
  login: (credentials) => api.post('/auth/login', credentials),
  register: (data) => api.post('/auth/register', data),
  logout: () => api.get('/auth/logout')
}

export const checkinAPI = {
  doCheckin: () => api.post('/checkin'),
  getHistory: () => api.get('/checkin/history')
}

export const badgesAPI = {
  getList: () => api.get('/badges'),
  getApiList: () => api.get('/api/badges')
}

export const dashboardAPI = {
  getIndex: () => api.get('/dashboard')
}

export const profileAPI = {
  get: () => api.get('/profile'),
  update: (data) => api.post('/profile', data)
}

export default api
```

- [ ] **Step 3: 创建用户 Store**

```javascript
// frontend/src/stores/user.js
import { defineStore } from 'pinia'
import { authAPI } from '@/api'

export const useUserStore = defineStore('user', {
  state: () => ({
    id: localStorage.getItem('user_id') || null,
    username: localStorage.getItem('username') || '',
    hometown: '',
    currentCity: '',
    familyRole: '妈妈',
    nickname: '',
    toneStyle: '唠叨型'
  }),

  getters: {
    isAuthenticated: (state) => !!state.id
  },

  actions: {
    async login(credentials) {
      const response = await authAPI.login(credentials)
      if (response.success) {
        this.id = response.user_id
        this.username = response.username
        localStorage.setItem('user_id', response.user_id)
        localStorage.setItem('username', response.username)
      }
      return response
    },

    async register(data) {
      const response = await authAPI.register(data)
      return response
    },

    logout() {
      this.id = null
      this.username = ''
      localStorage.removeItem('user_id')
      localStorage.removeItem('username')
    },

    updateProfile(data) {
      this.hometown = data.hometown
      this.currentCity = data.current_city
      this.familyRole = data.family_role
      this.nickname = data.nickname
      this.toneStyle = data.tone_style
    }
  }
})
```

- [ ] **Step 4: 创建勋章 Store**

```javascript
// frontend/src/stores/badges.js
import { defineStore } from 'pinia'
import { badgesAPI } from '@/api'

export const useBadgesStore = defineStore('badges', {
  state: () => ({
    list: [],
    categories: {
      time: { name: '时间里程碑', badges: [] },
      streak: { name: '签到连续', badges: [] },
      special: { name: '特殊成就', badges: [] }
    }
  }),

  getters: {
    earnedCount: (state) => state.list.length,
    earnedIds: (state) => new Set(state.list.map(b => b.id))
  },

  actions: {
    async fetchBadges() {
      const data = await badgesAPI.getApiList()
      this.list = data
    },

    addBadge(badge) {
      if (!this.earnedIds.has(badge.id)) {
        this.list.push(badge)
      }
    }
  }
})
```

- [ ] **Step 5: 创建签到 Store**

```javascript
// frontend/src/stores/checkin.js
import { defineStore } from 'pinia'
import { checkinAPI } from '@/api'

export const useCheckinStore = defineStore('checkin', {
  state: () => ({
    todayChecked: false,
    streak: 0,
    totalDays: 0,
    lastCheckin: null,
    newBadges: []
  }),

  actions: {
    async doCheckin() {
      const result = await checkinAPI.doCheckin()
      if (result.success) {
        this.todayChecked = true
        this.streak = result.streak
        this.totalDays = result.total_days
        this.newBadges = result.new_badges || []
        return result
      }
      throw new Error('签到失败')
    },

    resetNewBadges() {
      this.newBadges = []
    }
  }
})
```

- [ ] **Step 6: 提交**

```bash
cd /Users/old_people/test/GoHome
git add frontend/src/router frontend/src/stores frontend/src/api
git commit -m "feat: add Vue Router, Pinia stores, and API layer"
```

---

### Task 11: 创建签到页面组件

**Files:**
- Create: `frontend/src/views/Checkin.vue`
- Create: `frontend/src/components/FamilyGreeting.vue`
- Create: `frontend/src/components/BadgeCard.vue`
- Create: `frontend/src/components/BadgePopup.vue`

- [ ] **Step 1: 创建家人问候组件**

```vue
<!-- frontend/src/components/FamilyGreeting.vue -->
<template>
  <div class="family-greeting">
    <div class="greeting-header">
      <span class="family-role">{{ familyRole }}</span>
      <span class="greeting-dialect">{{ dialect }}</span>
    </div>
    <p class="greeting-content">"{{ content }}"</p>
  </div>
</template>

<script setup>
defineProps({
  familyRole: { type: String, default: '妈妈' },
  dialect: { type: String, default: '' },
  content: { type: String, required: true }
})
</script>

<style scoped>
.family-greeting {
  background: linear-gradient(135deg, #FFF5F5 0%, #FFF0E6 100%);
  border-radius: 16px;
  padding: 24px;
  margin: 20px 0;
  border: 2px solid #FFB6C1;
}

.greeting-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.family-role {
  font-size: 18px;
  font-weight: bold;
  color: #F5A623;
}

.greeting-dialect {
  font-size: 14px;
  color: #888;
}

.greeting-content {
  font-size: 18px;
  line-height: 1.6;
  color: #333;
  font-style: italic;
}
</style>
```

- [ ] **Step 2: 创建勋章卡片组件**

```vue
<!-- frontend/src/components/BadgeCard.vue -->
<template>
  <div class="badge-card" :class="{ locked: !earned }" :title="description">
    <div class="badge-icon">{{ icon }}</div>
    <div class="badge-info">
      <h3>{{ name }}</h3>
      <p class="badge-description">{{ description }}</p>
      <p v-if="earned" class="badge-earned">获得时间：{{ earnedAt }}</p>
      <p v-else class="badge-locked">🔒 未解锁</p>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  id: String,
  name: String,
  description: String,
  earned: Boolean,
  earnedAt: String
})

const icon = computed(() => {
  if (!props.earned) return '🔒'
  const parts = props.name.split(' ')
  return parts[0] || '⭐'
})
</script>

<style scoped>
.badge-card {
  background: #fff;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
  transition: transform 0.2s;
  cursor: pointer;
}

.badge-card:hover {
  transform: translateY(-4px);
}

.badge-card.locked {
  opacity: 0.6;
  filter: grayscale(80%);
}

.badge-icon {
  font-size: 48px;
  text-align: center;
  margin-bottom: 10px;
}

.badge-info h3 {
  font-size: 16px;
  margin-bottom: 8px;
}

.badge-description {
  font-size: 14px;
  color: #666;
  margin-bottom: 8px;
}

.badge-earned {
  font-size: 12px;
  color: #98D8AA;
}

.badge-locked {
  font-size: 12px;
  color: #888;
}
</style>
```

- [ ] **Step 3: 创建勋章弹窗组件**

```vue
<!-- frontend/src/components/BadgePopup.vue -->
<template>
  <div class="popup-overlay" v-if="show" @click="close">
    <div class="popup-content" @click.stop>
      <div class="confetti">🎉</div>
      <h2>恭喜解锁新勋章!</h2>
      <div class="badge-showcase">
        <BadgeCard
          v-for="badge in badges"
          :key="badge.id"
          :id="badge.id"
          :name="badge.name"
          :description="badge.description"
          :earned="true"
          :earned-at="new Date().toISOString().slice(0, 10)"
        />
      </div>
      <button class="btn btn-primary" @click="close">确定</button>
    </div>
  </div>
</template>

<script setup>
import BadgeCard from './BadgeCard.vue'

defineProps({
  show: Boolean,
  badges: Array
})

const emit = defineEmits(['close'])
const close = () => emit('close')
</script>

<style scoped>
.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.popup-content {
  background: linear-gradient(135deg, #FFF9E6 0%, #FFF0E6 100%);
  border-radius: 20px;
  padding: 32px;
  max-width: 500px;
  width: 90%;
  text-align: center;
  border: 3px solid #F5A623;
}

.confetti {
  font-size: 48px;
  margin-bottom: 16px;
}

.popup-content h2 {
  color: #333;
  margin-bottom: 24px;
}

.badge-showcase {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 16px;
  margin-bottom: 24px;
}

.btn {
  padding: 12px 32px;
  font-size: 16px;
}
</style>
```

- [ ] **Step 4: 创建签到页面**

```vue
<!-- frontend/src/views/Checkin.vue -->
<template>
  <div class="checkin-page">
    <h1>每日签到</h1>
    <p class="subtitle">记录离家的每一天，珍藏每一份思念</p>

    <div v-if="alreadyChecked" class="already-checked">
      <div class="check-icon">✅</div>
      <h2>今天已经签到过了</h2>
      <p>明天记得再来哦~</p>
      <router-link to="/dashboard" class="btn btn-primary">返回首页</router-link>
    </div>

    <div v-else class="checkin-form">
      <button
        class="btn btn-primary btn-large"
        @click="handleCheckin"
        :disabled="loading"
      >
        {{ loading ? '签到中...' : '立即签到' }}
      </button>
    </div>

    <!-- 家人问候 -->
    <FamilyGreeting
      v-if="greeting"
      :family-role="greeting.family_role"
      :dialect="greeting.dialect"
      :content="greeting.content"
    />

    <!-- 新解锁勋章弹窗 -->
    <BadgePopup
      :show="showPopup"
      :badges="newBadges"
      @close="closePopup"
    />
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useCheckinStore } from '@/stores/checkin'
import { useBadgesStore } from '@/stores/badges'
import FamilyGreeting from '@/components/FamilyGreeting.vue'
import BadgePopup from '@/components/BadgePopup.vue'

const checkinStore = useCheckinStore()
const badgesStore = useBadgesStore()

const loading = ref(false)
const alreadyChecked = ref(false)
const greeting = ref(null)
const showPopup = ref(false)
const newBadges = ref([])

const handleCheckin = async () => {
  loading.value = true
  try {
    const result = await checkinStore.doCheckin()

    // 显示家人问候
    greeting.value = {
      family_role: result.family_role || '妈妈',
      dialect: result.dialect || '',
      content: result.quote_content
    }

    // 显示新勋章弹窗
    if (result.new_badges && result.new_badges.length > 0) {
      newBadges.value = result.new_badges
      showPopup.value = true
    }

    alreadyChecked.value = true
  } catch (error) {
    alert('签到失败：' + error.message)
  } finally {
    loading.value = false
  }
}

const closePopup = () => {
  showPopup.value = false
  checkinStore.resetNewBadges()
}
</script>

<style scoped>
.checkin-page {
  max-width: 600px;
  margin: 0 auto;
  text-align: center;
  padding: 40px 20px;
}

h1 {
  font-size: 28px;
  margin-bottom: 8px;
}

.subtitle {
  color: #888;
  margin-bottom: 40px;
}

.btn-large {
  padding: 16px 48px;
  font-size: 20px;
}

.already-checked {
  padding: 40px;
}

.check-icon {
  font-size: 64px;
  margin-bottom: 16px;
}

.checkin-form {
  padding: 40px;
}
</style>
```

- [ ] **Step 5: 提交**

```bash
cd /Users/old_people/test/GoHome
git add frontend/src/views/Checkin.vue frontend/src/components/
git commit -m "feat: create Checkin view and related components"
```

---

### Task 12: 创建勋章墙和仪表盘页面

**Files:**
- Create: `frontend/src/views/Badges.vue`
- Create: `frontend/src/views/Dashboard.vue`
- Create: `frontend/src/views/Login.vue`
- Create: `frontend/src/views/Register.vue`
- Create: `frontend/src/views/Profile.vue`

- [ ] **Step 1: 创建勋章墙页面**

```vue
<!-- frontend/src/views/Badges.vue -->
<template>
  <div class="badges-page">
    <h1>我的勋章墙</h1>
    <p class="subtitle">记录你的每一个成长瞬间</p>

    <div v-for="(category, key) in categories" :key="key" class="badge-section">
      <h2>{{ category.name }}</h2>
      <div class="badge-grid">
        <BadgeCard
          v-for="badge in category.badges"
          :key="badge.id"
          :id="badge.id"
          :name="badge.name"
          :description="badge.description"
          :earned="badge.earned"
          :earned-at="badge.earned_at"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useBadgesStore } from '@/stores/badges'
import BadgeCard from '@/components/BadgeCard.vue'

const badgesStore = useBadgesStore()
const categories = ref({})

onMounted(async () => {
  await badgesStore.fetchBadges()
  // 从 API 返回的数据按分类组织
  // 这里简化处理，实际需要根据后端 API 调整
})
</script>

<style scoped>
.badges-page {
  max-width: 960px;
  margin: 0 auto;
  padding: 20px;
}

.badge-section {
  margin-bottom: 40px;
}

.badge-section h2 {
  color: #333;
  margin-bottom: 20px;
}

.badge-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 20px;
}
</style>
```

- [ ] **Step 2: 创建登录页面**

```vue
<!-- frontend/src/views/Login.vue -->
<template>
  <div class="login-page">
    <div class="login-card card">
      <h1>登录</h1>
      <form @submit.prevent="handleLogin">
        <div class="form-group">
          <label for="username">用户名</label>
          <input
            id="username"
            v-model="username"
            type="text"
            required
            placeholder="请输入用户名"
          />
        </div>
        <div class="form-group">
          <label for="password">密码</label>
          <input
            id="password"
            v-model="password"
            type="password"
            required
            placeholder="请输入密码"
          />
        </div>
        <div class="form-group checkbox-group">
          <label>
            <input type="checkbox" v-model="remember" />
            记住我
          </label>
        </div>
        <button type="submit" class="btn btn-primary btn-block">登录</button>
      </form>
      <p class="form-footer">
        还没有账号？<router-link to="/register">立即注册</router-link>
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'

const router = useRouter()
const userStore = useUserStore()

const username = ref('')
const password = ref('')
const remember = ref(false)

const handleLogin = async () => {
  try {
    const result = await userStore.login({
      username: username.value,
      password: password.value,
      remember: remember.value
    })

    if (result.success) {
      router.push('/dashboard')
    }
  } catch (error) {
    alert('登录失败：' + error.message)
  }
}
</script>

<style scoped>
.login-page {
  min-height: 80vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

.login-card {
  max-width: 400px;
  width: 100%;
  padding: 32px;
}

.login-card h1 {
  text-align: center;
  margin-bottom: 24px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
}

.form-group input[type="text"],
.form-group input[type="password"] {
  width: 100%;
  padding: 12px;
  border: 2px solid #E8E8E8;
  border-radius: 8px;
  font-size: 16px;
  transition: border-color 0.2s;
}

.form-group input:focus {
  outline: none;
  border-color: #F5A623;
}

.checkbox-group label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.btn-block {
  width: 100%;
  padding: 14px;
  font-size: 16px;
}

.form-footer {
  text-align: center;
  margin-top: 20px;
  color: #888;
}
</style>
```

- [ ] **Step 3: 提交**

```bash
cd /Users/old_people/test/GoHome
git add frontend/src/views/Badges.vue frontend/src/views/Login.vue
git commit -m "feat: create Badges and Login views"
```

---

## 最终检查

- [ ] 运行所有测试

```bash
cd /Users/old_people/test/GoHome
pixi run pytest tests/ -v
```

- [ ] 启动应用手动验证

```bash
cd /Users/old_people/test/GoHome
# 后端
pixi run dev

# 前端 (新终端)
cd frontend
npm run dev
```

访问 http://localhost:3000 (前端开发服务器)

---

**计划结束**
