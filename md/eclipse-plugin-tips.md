
### 打开文件

```java
import org.eclipse.core.resources.IFile;
import org.eclipse.core.resources.IFolder;
import org.eclipse.core.resources.IProject;
import org.eclipse.core.resources.IResource;
import org.eclipse.core.resources.IWorkspaceRoot;
import org.eclipse.core.resources.ResourcesPlugin;
import org.eclipse.ui.IEditorInput;
import org.eclipse.ui.IWorkbenchPage;
import org.eclipse.ui.PartInitException;
import org.eclipse.ui.PlatformUI;
import org.eclipse.ui.ide.IDE;


IWorkspaceRoot root = ResourcesPlugin.getWorkspace().getRoot();
   IWorkbenchPage page = PlatformUI.getWorkbench().getActiveWorkbenchWindow().getActivePage();
   IEditorInput input = page.getActiveEditor()
     .getEditorInput();
   IResource activeResource = input.getAdapter(IResource.class);
   IProject iproject = activeResource.getProject();
   IFolder iFolder = iproject.getFolder("src/demo");
   IFile iFile = iFolder.getFile("test.java");

   try {
    IDE.openEditor(page, iFile);
   } catch (PartInitException e) {
    e.printStackTrace();
   }

```

### 获取图标的工具类

```java
package com.leaderli.li.flow.util;

import java.net.MalformedURLException;
import java.net.URL;

import org.eclipse.jface.resource.ImageDescriptor;
import org.eclipse.jface.resource.ImageRegistry;
import org.eclipse.swt.graphics.Image;

//插件的启动类，一般自动生成
import com.leaderli.li.flow.Activator;

public class ImageUtil {
 public static final ImageRegistry IMAGE_REGISTRY = Activator.getDefault().getImageRegistry();

 public static Image getImage(String name) {

  getImageDescriptor(name);
  return IMAGE_REGISTRY.get(name);
 }

 public static ImageDescriptor getImageDescriptor(String name) {

  ImageDescriptor imageDescriptor = IMAGE_REGISTRY.getDescriptor(name);
  Image image = IMAGE_REGISTRY.get(name);
  if (imageDescriptor == null) {
   try {
    imageDescriptor = ImageDescriptor
      .createFromURL(new URL(Activator.getDefault().getBundle().getEntry("/"), "icon/" + name));
    IMAGE_REGISTRY.put(name, imageDescriptor);
   } catch (MalformedURLException e) {
    e.printStackTrace();
   }
  }
  return imageDescriptor;
 }
}

```


### 读取插件下的资源

```java
import com.leaderli.visual.editor.Activator;

public static void copyFileFromPluginToProject(IProject project, String from, String to, IProgressMonitor monitor) throws Exception 
	// form   resource/pom.xml
	URL fromURL = Activator.getDefault().getBundle().getResource(from);
	// toFile   pom.xml
	IFile toFile = project.getFile(to);
	//将插件下的pom.xml拷贝到项目根目录下
	toFile.create(fromStream, true, monitor);

}
```
### 进度条
参考 [Eclipse Jobs and Background Processing - Tutorial](https://www.vogella.com/tutorials/EclipseJobs/article.html)

```java
import java.util.concurrent.TimeUnit;

import org.eclipse.core.runtime.IProgressMonitor;
import org.eclipse.core.runtime.IStatus;
import org.eclipse.core.runtime.Status;
import org.eclipse.core.runtime.SubMonitor;
import org.eclipse.core.runtime.jobs.Job


Job job = new Job("create project : " + projectName) {

		@Override
		protected IStatus run(IProgressMonitor monitor) {
			try {
				//将任务分为100个
				SubMonitor subMonitor = SubMonitor.convert(monitor, 100);
				for (int i = 0; i < 10; i++) {
					//完成10个
					subMonitor.split(10);
					monitor.setTaskName("create project:" + i);
					// 设定为还剩70个
					// subMonitor.setWorkRemaining(70);
					TimeUnit.SECONDS.sleep(1);
				}

			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			return new Status(IStatus.OK, Activator.PLUGIN_ID, Messages.status_OK);
		}
};
job.schedule();
```

###  创建一个maven项目
在创建project的时候，将nature和buildSpec中添加maven的支持

```java
import org.eclipse.core.resources.ICommand;
import org.eclipse.core.resources.IProject;
import org.eclipse.core.resources.IProjectDescription;
import org.eclipse.core.runtime.NullProgressMonitor;
import org.eclipse.ui.IWorkbenchWindow;


private static void addMavenNature(IProject project, IProgressMonitor monitor) throws CoreException {


	IProjectDescription description = project.getDescription();

	String[] newNatures = new String[2];

	newNatures[0] = JavaCore.NATURE_ID;// 添加 java
	newNatures[1] = "org.eclipse.m2e.core.maven2Nature";// 添加 maven

	description.setNatureIds(newNatures);

	project.setDescription(description, monitor);

	ICommand command = description.newCommand();
	command.setBuilderName(JavaCore.BUILDER_ID); // 添加java
	
	command = description.newCommand();
	command.setBuilderName("org.eclipse.m2e.core.maven2Builder"); // 添加 maven
}  
```

也需要在设置classpath的内容

```xml
<?xml version="1.0" encoding="UTF-8"?>
<classpath>
	<classpathentry kind="src" output="target/test-classes" path="src/test/java">
		<attributes>
			<attribute name="optional" value="true"/>
			<attribute name="maven.pomderived" value="true"/>
		</attributes>
	</classpathentry>
	<classpathentry kind="src" path="src/main/java"/>
	<classpathentry kind="con" path="org.eclipse.jdt.launching.JRE_CONTAINER/org.eclipse.jdt.internal.debug.ui.launcher.StandardVMType/jdk1.8.0_281">
		<attributes>
			<attribute name="maven.pomderived" value="true"/>
		</attributes>
	</classpathentry>
	<classpathentry kind="con" path="org.eclipse.m2e.MAVEN2_CLASSPATH_CONTAINER">
		<attributes>
			<attribute name="maven.pomderived" value="true"/>
		</attributes>
	</classpathentry>
	<classpathentry kind="output" path="target/classes"/>
</classpath>
```

###  创建WizardPage时用到的一些方法
```java
this.setMessage("Select a project.", IMessageProvider.WARNING); // 2
```