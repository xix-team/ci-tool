# 持续集成工具库

## 使用说明

解析开发人员提交的源码分支的引用名称(ref),得出目标环境(env),

在构建前把源码中的环境参数[@ENV@|@VERSION@|@BUILD@]替换,在构建后,把可运行程序从源目录(from)发布到目标环境(target-env)的目标目录(to)

## ./detect

解析分支获取服务环境

## ./npm-publish

包组件部署

### ./deploy

发布脚本

``` bash
deploy --from=<from> --to=<to> --ref=<ref> --target-dev=<target> --target-sit=<target> --target-uat=<target> --target-<target>=<target> [options] <project>
```

参数说明
```
--from=<from>             必填,源目录,结尾不需要带/               eg: /var/lib/jenkins/workspace/project
--to=<to>                 必填,目标目录,结尾不需要带/             eg: /usr/src/project
--ref=<ref>               当前要发布的项目的分支名称,当分支为master、release-*、tag的时候会促发git发布 eg: master release/* hotfix/* release-v1 release-v2 develop

target是指目标服务器,值为<local|[user@]host|git>可以设置多个,用逗号分隔 eg: lijian@10.8.82.11,lijian@10.8.64.35
--target-dev=<target>     dev环境 feature   -> dev
--target-sit=<target>     sit环境 develop   -> sit
--target-uat=<target>     uat环境 master/*,release/*,hotfix/* -> uat
--target-prd=<target>     prd环境 tag/x.x.x -> dist
--target-branch=<branch>  提交到target的git的指定分支,默认为master

--deploy-before=<shell>   发布前执行的远程脚本
--deploy-after=<shell>    发布后执行的远程脚本

--build=<build>           编译号,默认为当前时间,eg:20160501103020
--clean                   保持from和to内容完全一致,默认为false
--verbose                 显示发布详情
--replace-dir=<dir>       替换参数的文件的目录,默认为源目录的根目录,可设置为子目录,替换的环境参数为[@ENV@|@VERSION@|@BUILD@]
--replace-suffix=<suffix> 替换参数的文件后缀,默认:ini|conf|htm|html|xhtml|ftl|tpl|css|js|json|manifest
--help                    help
```

## 测试流程

一般流程是DEV->SIT->UAT->PRD，系统测试通过后交付用户测试，用户测试通过后提交至生产机．如果每次用户测试不通过，就得从SIT迭代开始

* SIT:System   Integration   TestCase（系统集成测试，即内部测试）
根据用例描述测试每一个场景，优化系统性能，提交数据库性能excution plan给DBA review。对系统进行压力测试（必要情况下提交到APCC的压力测试组进行测试）。里程碑：完成内部测试报告和得到DBA的上线批准。

* UAT : User   Acceptance   Test（用户验收测试，即用户测试）
用户根据用例描述测试每一个场景，反馈系统issue。开发人员基于issue对系统影响和对业务impact判断，适当的修正系统或记录业务需求，根据业务优先等级，集成进下一个演进阶段。 里程碑：UAT Sign off。用户签收当前系统功能。

* SIT和UAT两者区别：
从时间上看，UAT要在SIT后面，UAT测试要在系统测试完成后才开始。
从测试人员看，SIT由公司的测试员来测试，而UAT一般是由用户来测试。

## 服务环境

* dev  开发环境
* sit  系统集成测试环境
* uat  用户验收测试环境
* prd  生产环境


## 部署流程

* develop   -> dev
提交develop分支,jenkins自动部署到dev
* sit/*       -> sit
提交sit分支,jenkins自动部署到sit
* uat/*       -> uat
提交uat分支,jenkins自动部署到uat
* release/* -> uat
提交release分支,jenkins自动部署到uat
* hotfix/*  -> uat
提交hotfix分支,jenkins自动部署到uat
* master/*  -> uat
提交master分支,jenkins自动部署到uat
* tags/x.x.x -> prd/dist
在master分支上打标签,jenkins自动部署到prd/dist


## jenkins配置示例

``` bash

project=${JOB_NAME##*/}

git_commit_id=$(git log -1 --pretty=format:"%h")

env=$(/opt/ci-tool/detect $branch)

if test "$env" != "" ; then
npm run build-$env
bash -e /opt/ci-tool/deploy --from=$WORKSPACE/dist --to=/home/www/@ENV@/$project --target-dev="$DEV" --target-sit="$SIT" --target-uat="$UAT"  --target-prd="default" --verbose --build=$git_commit_id --ref=$branch $project
fi

```


