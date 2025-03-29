-- 创建数据库
CREATE DATABASE IF NOT EXISTS social_media_analysis;
USE social_media_analysis;

-- 创建社交媒体平台表
CREATE TABLE social_media (
    media_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE
) COMMENT '存储社交媒体平台信息';

-- 创建用户表
CREATE TABLE user (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    media_id INT NOT NULL,
    username VARCHAR(40) NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    country_of_birth VARCHAR(50),
    country_of_residence VARCHAR(50),
    age INT,
    gender VARCHAR(20),
    is_verified BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (media_id) REFERENCES social_media(media_id),
    UNIQUE KEY (media_id, username) -- 用户名在特定平台上是唯一的
) COMMENT '存储社交媒体用户信息';

-- 创建帖子表
CREATE TABLE post (
    post_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    media_id INT NOT NULL,
    content TEXT NOT NULL,
    posted_time DATETIME NOT NULL,
    location_city VARCHAR(50),
    location_state VARCHAR(50),
    location_country VARCHAR(50),
    likes INT DEFAULT 0,
    dislikes INT DEFAULT 0,
    has_multimedia BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (user_id) REFERENCES user(user_id),
    FOREIGN KEY (media_id) REFERENCES social_media(media_id),
    UNIQUE KEY (user_id, media_id, posted_time) -- 同一用户在同一平台不能在同一时间发多个帖子
) COMMENT '存储社交媒体帖子';

-- 创建转发表
CREATE TABLE repost (
    repost_id INT AUTO_INCREMENT PRIMARY KEY,
    post_id INT NOT NULL,
    user_id INT NOT NULL,
    repost_time DATETIME NOT NULL,
    FOREIGN KEY (post_id) REFERENCES post(post_id),
    FOREIGN KEY (user_id) REFERENCES user(user_id)
) COMMENT '存储帖子的转发信息';

-- 创建研究机构表
CREATE TABLE institute (
    institute_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
) COMMENT '存储研究机构信息';

-- 创建项目表
CREATE TABLE project (
    project_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    manager_first_name VARCHAR(50) NOT NULL,
    manager_last_name VARCHAR(50) NOT NULL,
    institute_id INT NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    FOREIGN KEY (institute_id) REFERENCES institute(institute_id),
    CONSTRAINT chk_date CHECK (end_date >= start_date)
) COMMENT '存储分析项目信息';

-- 创建项目字段表
CREATE TABLE project_field (
    field_id INT AUTO_INCREMENT PRIMARY KEY,
    project_id INT NOT NULL,
    field_name VARCHAR(50) NOT NULL,
    FOREIGN KEY (project_id) REFERENCES project(project_id),
    UNIQUE KEY (project_id, field_name) -- 字段名在项目内是唯一的
) COMMENT '存储项目分析字段';

-- 创建项目-帖子关联表
CREATE TABLE project_post (
    project_post_id INT AUTO_INCREMENT PRIMARY KEY,
    project_id INT NOT NULL,
    post_id INT NOT NULL,
    FOREIGN KEY (project_id) REFERENCES project(project_id),
    FOREIGN KEY (post_id) REFERENCES post(post_id),
    UNIQUE KEY (project_id, post_id) -- 避免重复分析
) COMMENT '存储项目与帖子的关联';

-- 创建分析结果表
CREATE TABLE analysis_result (
    result_id INT AUTO_INCREMENT PRIMARY KEY,
    project_post_id INT NOT NULL,
    field_id INT NOT NULL,
    value VARCHAR(255),
    FOREIGN KEY (project_post_id) REFERENCES project_post(project_post_id),
    FOREIGN KEY (field_id) REFERENCES project_field(field_id),
    UNIQUE KEY (project_post_id, field_id) -- 每个帖子的每个字段只有一个结果
) COMMENT '存储分析结果';

-- 创建索引以优化查询性能
CREATE INDEX idx_post_media ON post(media_id);
CREATE INDEX idx_post_time ON post(posted_time);
CREATE INDEX idx_post_user ON post(user_id);
CREATE INDEX idx_user_names ON user(first_name, last_name);
CREATE INDEX idx_project_post_project ON project_post(project_id);
CREATE INDEX idx_project_post_post ON project_post(post_id);
CREATE INDEX idx_analysis_result_field ON analysis_result(field_id);