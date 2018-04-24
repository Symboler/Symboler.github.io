---
title: 使用reta开发简单的后台管理页面
date: 2018-04-24 11:02:33
tags:
---
## reta使用
### 功能模块开发流程总结
#### 以服务列表页面开发为例
1、分析布局
确定BasicLayout布局
2、定义router组件
现在routes文件夹中定义空组件Demo备用
    `

	import React, { Component } from 'react'
	class Demo extends Component {
  		constructor(props) {
    		super(props);
  		}

  		componentWillMount() {

  		}
  		render() {
    		return (
      			<div>
       			demo页面
      			</div>
    		)
  		}
	}

	export default Demo`

在nav.js中添加页面路由配置
    `

	{
        name: '测试页面',
        path: 'demo',
        component: dynamicWrapper(app, [], () => import('../routes/ServicesList')),
      }`

在headerMenu中添加菜单项
    `

		{
    		text: '测试页面',
    		path: '/demo',
    		useLink: true,
    		replace: false,
  		},
`
3、定义model组件
在models定义demo.js备用，并在nav中demo组件中绑定['demo']
    `

	export default {
  		namespace: 'demo',

  		state: {

  		},

  		effects: {

  		},

  		reducers: {

  		},

  		subscriptions: {

  		},
	}`

完成上面3步后，我们页面里面已经有空白的测试页面了
![](http://lxlimg.gz.bcebos.com/list01.png)


4、开始页面开发

1）分析页面
![](http://lxlimg.gz.bcebos.com/list.png)
服务列表页面可划分为四部分，
1：面包屑，已在layout中实现，不需要考虑；
2：searchpanel
3：左边siderbar
4: 右边table
注：对于实际项目，类似功能已抽离为组件形式，这里我还是不引用现成组件，从零开始开发，首先完成静态页面的搭建。

第一部分searchpanel
不解释直接上代码

    `render() {
  	 	const ctrlStyle = {
        	textAlign: 'right'
      	};
    	return (
       	<Form
       		className={classNames(Styles.searchPanel)}
      	>
        	<Row gutter={24}>
           		<Col span={8}>
          			<Form.Item
          				wrapperCol={{span:18}}
          			>
           				<Input placeholder={"请输入服务名称搜索"}/>
          			</Form.Item>
        		</Col>
        		<Col span={16} style={ctrlStyle}>
					 <Form.Item
					 >
                		<Button key='primary' type='primary' >搜索</Button>
                		<Button key='reset' style={{ marginLeft: 8 }}>重置</Button>
           			</Form.Item>
        		</Col>
        	</Row>
      	</Form>
    	)
  	}`

第二部分sider

    `//
		<Row gutter={24} style={{ margin: 0 }}>
          <Col span={4} key='child' style={{ paddingLeft: 0 }}>
						 <Menu
        			mode="inline"
        			defaultSelectedKeys={[ '-1' ]}
        			className={Styles.menu}
      			>
        			{
          			menuData.subfiles === undefined ? null :
          			menuData.subfiles.map(item => this.fileItem(item))
        			}
        		{
          			menuData.subfolders === undefined ? null:
          			menuData.subfolders.map(item => this.menuItem(item))
        			}
      			</Menu>
          </Col>
          <Col span={20} key='table' style={{ paddingRight: 0 }}>table</Col>
        </Row>`



    `fileItem(item) {
    	return (
      		<Menu.Item key={item.id}>{item.name}</Menu.Item>
    	)
  	}
  	menuItem (item) {
    	if (this.isSubfiles) {
      		if (item.subfolders && item.subfolders.length) {
        		return (
          			<SubMenu key={item.path} title={<span><span>{item.name}</span></span>}>
            		{
              			item.subfiles.map(v => this.fileItem(v))
            		}
            		{
              			item.subfolders.map(v => this.menuItem(v))
            		}
         			 </SubMenu>
        		)
      		} else if (item.subfiles && item.subfiles.length){
        		return (
          			<SubMenu key={item.path} title={<span><span>{item.name}</span></span>}>
            		{
              			item.subfiles.map(v => this.fileItem(v))
            		}
          			</SubMenu>
        		)
      		} else {
        		return <Menu.Item key={item.path} disabled>{item.name}</Menu.Item>
      		}
    		} else {
      		if (item.subfolders && item.subfolders.length) {
        		return (
          			<SubMenu onTitleClick={this.props.onClick} key={item.path} title={<span><span>{item.name}</span></span>}>
            		{
              		item.subfolders.length === 0 ? null :
              		item.subfolders.map(v => this.menuItem(v))
            		}
          			</SubMenu>
       		 )
      	} else {
        	return <Menu.Item key={item.path}>{item.name}</Menu.Item>
      	}
    	}
  	}`


    `//模拟数据

	 const menuData = {

      		"path":"/5",
      		"name":"api",
      		"subfolders":[
      			{ name: '所有服务', path: '-1'},
     			{"path":"/5/1064",
     			 	"name":"交通出行",
      			 	"code":"traffic",
      				"description":"绿色出行 低碳生活1",
      				"subfolders":[]
    			},
    			{"path":"/5/1006",
    				"name":"金融理财",
    				"code":"jinrong",
    				"description":"金融",
    				"subfolders":[
    					{"path":"/5/1006/1023",
    						"name":"农业市场",
    						"code":"farmmarket",
    						"description":"",
    						"subfolders":[
    						{"path":"/5/1006/1023/1024",
    							"name":"水果经济",
    							"code":"fruitEnco",
    							"description":"",
    							"subfolders":[]
  							}]
						}
				]},
				{"path":"/5/1151",
					"name":"我在东北玩泥巴",
					"code":"niba",
					"description":"我在大连没有家",
					"subfolders":[]},
				{"path":"/5/1007",
					"name":"便民生活",
					"code":"life",
					"description":"便民生活的",
					"subfolders":[]}]
			}`


第三部分：table，此处省略了模拟数据的定义
	`

	<Col span={20} key='table' style={{ paddingRight: 0 }}>
		<Table
            className={Styles.frontTable}
          	dataSource={tableData}
          	columns={this.listColums}
          	rowKey={record => record.id}
            span={12}
              />
          </Col>`


上面三部分主要完成了整个静态页面，看一下页面的效果

![](http://lxlimg.gz.bcebos.com/demo002.png)

接下来就是要分析页面内的逻辑，处理前后端交互的部分
按照我自己的习惯，一般是先主后次，先易后难。
涉及交互实际上也包含了搜索、侧边栏展示及选择、表单的展示。
显然表单是我们页面的最主要的部分，同时单独去处理表单的显示交互是比较简单的，如果去处理搜索功能，它会影响到表单的展示，反而是属于复杂的部分。所以我们先处理表单数据的渲染。
表单涉及三个交互：1、数据的动态获取；2、分页的获取；3、查看详情

在处理交互之前，先明确reta把所有数据的处理已经放在models中，通过props传递给组件，因此对应的数据需要在models里面定义并讲store与组件关联


1、数据的动态获取
    `

	//routers/Demo/index.js
	 componentWillMount() {
			this.props.dispatch({
        		type: `demo/fetchTotalList`,
      		});
  	}
	const { servicesList,dispatch} = this.props;
		...
	<Table
        className={Styles.frontTable}
        dataSource={servicesList.list}
        columns={this.listColums}
        rowKey={record => record.id}
        span={12}
      />
		...
	export default connect(state => ({
  		servicesList: state.demo.servicesList
	}))(Demo);

	//models/demo.js

	 state: {
  		servicesList: [],
  	},

	effects: {
  	 *fetchTotalList({ query }, { call, put }) {
      	const res = yield call(queryTotalList, query);//queryTotalList是一个fetch请求，可以本地写模拟数据
      	if (res.status == 'success') {
        	yield put({
          	type: 'saveTotalListData',
          	payload: res.data,
        	});
      	} else {
        	yield put({
          		type: 'saveTotalListData',
          		payload: [],
        	});
      	}
    	},
  	},

	reducers: {
  	 	saveTotalListData(state, { payload }) {
      		return {
        		...state,
        		servicesList: payload,
      		};
    	},
  	},`

到此简单的异步请求表单数据并并展示已经完成。

第二步：分页器设置
    `//routers/Demo/index.js

	 paginationChange = (page, pageSize) => {
    	const query = {
      		offset: pageSize * (page - 1),
      		limit: pageSize,
    	};
     	this.props.dispatch({
        	type: `demo/fetchTotalList`,
        	query: {
             	...query,
            	dirPath: this.state.dirPath,
            }
     	 });

  	};
	...

	const paginationData = {
      total: 0,
      total: servicesList.totalCount,
      current: servicesList.pageNum,
      pageSize: servicesList.limit,
      showSizeChanger: true,
      showQuickJumper: true,
      pageSizeOptions: ["10", "20", "50", "100"],
      onChange: this.paginationChange,
      onShowSizeChange: this.paginationChange,
      showTotal: (total) => (`共有${total}条记录`),
    };

	...
	<Table
        className={Styles.frontTable}
        dataSource={servicesList.list}
        columns={this.listColums}
        pagination={paginationData}
        rowKey={record => record.id}
        span={12}
              />`

到此第二步分页展示完成。

第三步是查看详情实现
这个很简单，只要在表单的列属性里配置链接即可
	`

	 this.listColums = [
      {
        title: '服务名称',
        dataIndex: 'abilityName',
        className: 'no-right-border',
        width: '20%',
      },
      {
        title: '部门名称',
        dataIndex: 'deptName',
        className: 'no-right-border',
        width: '20%',
      },
      {
        title: '所属目录',
        dataIndex: 'dirName',
        className: 'no-right-border',
        width: '10%',
      },
      {
        title: '分级',
        dataIndex: 'clsName',
        className: 'no-right-border',
        width: '10%',
      },
      {
        title: '服务简介',
        dataIndex: 'briefDescription',
        className: 'no-right-border',
      },
      {
        title: '操作',
        dataIndex: 'action',
        width: this.props.match.path.indexOf('/devAccess/applyService') > -1 ? 140 : 100,
        render: (text, record) => (
          <span>
            <Link
              to={this.renderDetailLink(record.id)}
            >
              查看详情
            </Link>
          </span>
        ),
      }
    ]
	 this.renderDetailLink = id => {
      const path = this.props.match.path;
      if(path.indexOf('/devAccess/applyService') > -1) {
        return `/devAccess/applyService/${this.props.match.params.appId}/serviceInfo/${id}`
      } else if (path.indexOf('/devAccess') > -1) {
        return `/devAccess/servicesList/serviceInfo/${id}`;
      } else if (path.indexOf('/pubAccess') > -1) {
        return `/pubAccess/servicesList/serviceDetails/${id}`
      } else {
        return `/servicesList/serviceInfo/${id}`;
      }
    }`

上面三步完成了表单涉及的交互（当然还有些细节需要去完善，比如增加loading状态）

接下来去完成左侧菜单栏的动态获取

    `//models中处理数据请求
	  namespace: 'demo',

  		state: {
  		servicesList: [],
   		menuData: {
      		subfolders: []
    	},
  	},

	*fetchMenuData(_, { call, put }) {
      const res = yield call(queryMenuData);
      if (res.status === 'success') {
        const subFolders = res.data.subfolders || [];
        res.data.subfolders = [ { name: '所有服务', path: '-1'}, ...subFolders ]
        yield put({
          type: 'saveMenuData',
          payload: res.data,
        });
      }
    },
	saveMenuData(state, { payload }) {
      return {
        ...state,
        menuData: payload,
      };
    },
	...
	//页面渲染前获取数据
	componentWillMount() {
			 this.setState({
      			fetchList: this.props.match.path.indexOf('pubAccess') > -1 ? 'fetchPubList' : 'fetchTotalList',
    		}, () =>{
      		this.props.dispatch({
        		type: `demo/${this.state.fetchList}`,
      		});
    });
      this.props.dispatch({
      	type: "demo/fetchMenuData",
    	});
  	}
	//处理菜单项的点击事件，获取对应的表单数据，需要注意的是SubMenu也需要绑定click事件
	 handleClickSider(e) {
    	const { dispatch, currentUser } = this.props;
    	this.setState({
      		dirPath: e.key
    	}, () => {
      	const query = {
        	dirPath: this.state.dirPath,
      	}
      	dispatch({
        	type: `demo/${this.state.fetchList}`,
        	query: e.key === '-1' ? null : query,
      	});
    	})
  	}
	const { servicesList,dispatch,menuData} = this.props;
	...
 	menuItem (item) {
   	 if (this.isSubfiles) {
      	if (item.subfolders && item.subfolders.length) {
        	return (
          		<SubMenu key={item.path} title={<span><span>{item.name}</span></span>}>
            {
              item.subfiles.map(v => this.fileItem(v))
            }
            {
              item.subfolders.map(v => this.menuItem(v))
            }
          </SubMenu>
        )
      } else if (item.subfiles && item.subfiles.length){
        return (
          <SubMenu key={item.path} title={<span><span>{item.name}</span></span>}>
            {
              item.subfiles.map(v => this.fileItem(v))
            }
          </SubMenu>
        )
      } else {
        return <Menu.Item key={item.path} disabled>{item.name}</Menu.Item>
      }
    } else {
      if (item.subfolders && item.subfolders.length) {
        return (
          <SubMenu onTitleClick={this.handleClickSider.bind(this)} key={item.path} title={<span><span>{item.name}</span></span>}>
            {
              item.subfolders.length === 0 ? null :
              item.subfolders.map(v => this.menuItem(v))
            }
          </SubMenu>
        )
      } else {
        return <Menu.Item key={item.path}>{item.name}</Menu.Item>
      }
    }
  	}
	 <Menu
		onClick={this.handleClickSider.bind(this)}
        mode="inline"
       	defaultSelectedKeys={[ '-1' ]}
       	className={Styles.menu}
      	>
        {
          	menuData.subfiles === undefined ? null :
          	menuData.subfiles.map(item => this.fileItem(item))
        }
        {
          	menuData.subfolders === undefined ? null:
          	menuData.subfolders.map(item => this.menuItem(item))
        }
      	</Menu>`

到此左边菜单栏的交互也完成

最后一步就是搜索栏，这部分需要对ant的form组件熟悉掌握，form.create以及this.props.form自带api的使用

    `class SearchPanel extends Component {
	   handleSubmit = (e) => {
    	e && e.preventDefault && e.preventDefault();
 	 	this.props.form.validateFields((err, values) => {
      		if (err === null && this.props.fetchDataFunc) {
        		this.props.fetchDataFunc(values);
      		}
    	});

  		};
		handleReset = () => {
    		this.props.form.resetFields();
    		if (this.props.fetchDataFunc) {
      			this.props.fetchDataFunc();
    		}
  		};
  	render(){
  	 	const ctrlStyle = {
        	textAlign: 'right'
      	};
      	const { getFieldDecorator } = this.props.form;
       	const name="name"
       	const options= [
        	{
          	type: 'input',
          	name: 'name',
          	placeholder: '请输入服务名称搜索',
        	},
      	]

  		return (<Form
  			className={classNames(Styles.searchPanel)}
  			onSubmit={this.handleSubmit}
  			>
  			<Row gutter={24}>
  				<Col span={8}>
  					<Form.Item
  						wrapperCol={{span:18}}
  						name={name}
  					>  {
              		getFieldDecorator(name, options)(<Input placeholder={"请输入服务名称搜索"}/>)
            		}

  					</Form.Item>
  				</Col>
  				<Col span={16} style={ctrlStyle}>
  				<Form.Item
  				>
  				<Button key='primary' type='primary' htmlType='submit'>搜索</Button>
  				<Button key='reset' style={{ marginLeft: 8 }} onClick={this.handleReset}>重置</Button>
  				</Form.Item>
  				</Col>
  			</Row>
  		</Form>)
  		}
	}
	SearchPanel = Form.create({})(SearchPanel);
`

到此我们完成了一个常见的表单展示页面，包含search+sider+table

总结一下过程中的注意的问题点
1、整体布局需优先考虑好，采用antd的栅格来完成
2、searchPanel是对From组件的封装，需熟悉掌握Form.create的使用以及this.props.form的api
3、侧边栏即menu组件的使用
4、table组件的使用，主要就是dataSource、columns、pagination等属性的定义，别忘了rowKey
5、所有状态均在models中处理，通过props传入view组件

最后需要说明的是在实际项目中，这种功能模块的页面会有很多，所以通常会把searchPanel和sider分别抽离成一级组件；table+searchPanel+sider抽离为二级组件，通过属性配置来实现不同场景的展示与交互。
