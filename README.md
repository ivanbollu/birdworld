这是一个鸟类照片分享网站

1，网友可以上传自己自己拍摄鸟类的图片，  
2，同时可以选择鸟类的名称和所属分类，  
3，会员系统，注册会员后上传，  
4，加上地图显示，上传的时候，可以选择地图上一个点，在首页显示一副完整的地图，在上传过的地点，显示曾经上传过图片的缩略图。

完整配置步骤
使用 supabase.com+ vercel.com

第一步：创建 Supabase 项目
访问 supabase.com 注册/登录

点击 New Project

填写信息：

Name: bird-watching（可自定义）

Database Password: 设置一个强密码

Region: 选择离您最近的区域

等待项目创建（约2分钟）

第二步：1. 数据库表 (bird_observations)
在 Supabase SQL Editor 中执行以下 SQL：

sql
-- 创建观测记录表
CREATE TABLE bird_observations (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID NOT NULL,
  user_name TEXT NOT NULL,
  bird_name TEXT NOT NULL,
  category TEXT NOT NULL,
  image_url TEXT NOT NULL,
  lat DOUBLE PRECISION NOT NULL,
  lng DOUBLE PRECISION NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 创建索引提升查询性能
CREATE INDEX idx_observations_user_id ON bird_observations(user_id);
CREATE INDEX idx_observations_location ON bird_observations(lat, lng);
CREATE INDEX idx_observations_created_at ON bird_observations(created_at DESC);
2. 存储桶 (用于存放图片)
在 Supabase Storage 中：

创建名为 birdphotos 的存储桶

设置为 公开 (Public) 以便图片可以被所有人查看

配置存储策略允许用户上传：

sql
-- 允许已登录用户上传图片
CREATE POLICY "允许用户上传图片"
ON storage.objects
FOR INSERT
TO authenticated
WITH CHECK (bucket_id = 'birdphotos');

-- 允许公开查看图片
CREATE POLICY "允许公开查看图片"
ON storage.objects
FOR SELECT
TO public
USING (bucket_id = 'birdphotos');
3. 配置 Row Level Security (RLS)
sql
-- 启用 RLS
ALTER TABLE bird_observations ENABLE ROW LEVEL SECURITY;

-- 允许任何人查看观测记录
CREATE POLICY "允许公开查看观测记录"
ON bird_observations
FOR SELECT
TO public
USING (true);

-- 允许已登录用户插入记录
CREATE POLICY "允许登录用户发布观测"
ON bird_observations
FOR INSERT
TO authenticated
WITH CHECK (true);

第四步：获取 API 密钥
点击左侧 ⚙️ Project Settings

点击 API

复制两个值：

Project URL（类似 https://xxxxx.supabase.co）

anon public 密钥

第五步：配置网页
将之前代码中的这两行替换成您的实际值：

javascript
const SUPABASE_URL = 'https://你的项目ID.supabase.co';
const SUPABASE_ANON_KEY = '你的anon密钥';
第六步：测试运行
将完整代码保存为 index.html

双击用浏览器打开，或部署到任意静态服务器

注册账号 → 上传图片 → 点击地图选点 → 发布

💡 提示
图片上传：图片会存到 Supabase Storage，完全免费（1GB 空间）

用户系统：使用 Supabase Auth，邮箱注册即可

地图数据：经纬度存数据库，每次加载自动显示所有历史标记

如果遇到任何错误，请把浏览器控制台（F12）的红字报错发给我，我帮您排查！

https://chat.deepseek.com/a/chat/s/af765482-4f6a-44e6-a12d-c8f2a78a6a59
