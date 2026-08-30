安装langchain
pip install langchain==0.3.7
pip install langchain langchain-openai dotenv
查看已经安装的langchain版本
pip list |grep lang

删除langchain
pip uninstall langchain-text-splitters langgraph-checkpoint langgraph-prebuilt langgraph-sdk langsmith

安装最新兼容版本:
pip install "langsmith<0.2.0,>=0.1.17"
