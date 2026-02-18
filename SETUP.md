# Supabase 配置说明

## 第一步：创建 Supabase 项目

1. 访问 https://supabase.com
2. 点击 "Start your project" 注册账号
3. 登录后点击 "New Project"
4. 填写项目信息：
   - Name: `todolist`
   - Database Password: 设置一个强密码（记住它）
   - Region: 选择最近的区域（如 Singapore）
5. 点击 "Create new project"，等待项目创建完成（约2分钟）

## 第二步：获取 API 密钥

项目创建后：
1. 点击左侧菜单 "Settings" (齿轮图标)
2. 点击 "API"
3. 找到以下信息：
   - **Project URL**: `https://xxxxx.supabase.co`
   - **anon public**: `eyJhbGc...` (一长串字符)
4. 复制这两个值备用

## 第三步：创建数据库表

1. 点击左侧菜单 "SQL Editor"
2. 点击 "New query"
3. 粘贴以下 SQL 代码并点击 "Run":

```sql
-- 创建 profiles 表（用户信息）
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  email TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 创建 todos 表（待办事项）
CREATE TABLE todos (
  id BIGSERIAL PRIMARY KEY,
  text TEXT NOT NULL,
  done BOOLEAN DEFAULT FALSE,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 启用行级安全策略（RLS）
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE todos ENABLE ROW LEVEL SECURITY;

-- profiles 表权限策略
CREATE POLICY "用户可以查看所有 profiles"
  ON profiles FOR SELECT
  USING (true);

CREATE POLICY "用户只能插入自己的 profile"
  ON profiles FOR INSERT
  WITH CHECK (auth.uid() = id);

CREATE POLICY "用户只能更新自己的 profile"
  ON profiles FOR UPDATE
  USING (auth.uid() = id);

-- todos 表权限策略
CREATE POLICY "用户可以查看所有 todos"
  ON todos FOR SELECT
  USING (true);

CREATE POLICY "用户可以插入自己的 todos"
  ON todos FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "用户只能更新自己的 todos"
  ON todos FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "用户只能删除自己的 todos"
  ON todos FOR DELETE
  USING (auth.uid() = user_id);
```

4. 确认执行成功（右下角显示 "Success"）

## 第四步：配置网站

1. 打开 `index.html` 文件
2. 找到第 528-529 行：
```javascript
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

3. 替换为您的实际值：
```javascript
const SUPABASE_URL = 'https://xxxxx.supabase.co';  // 您的 Project URL
const SUPABASE_ANON_KEY = 'eyJhbGc...';  // 您的 anon public key
```

4. 保存文件

## 第五步：测试

1. 在浏览器打开 `index.html`
2. 点击 "注册" 创建账号
3. 填写姓名、邮箱、密码注册
4. 登录后即可使用

## 常见问题

**Q: 注册时提示邮箱已存在？**
A: 切换到登录模式，用该邮箱登录即可

**Q: 无法连接到 Supabase？**
A: 检查 SUPABASE_URL 和 SUPABASE_ANON_KEY 是否正确复制

**Q: 任务无法保存？**
A: 确保执行了第三步的 SQL 语句创建了数据库表
