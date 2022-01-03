## 概述
可能需要的依赖

```java
org.eclipse.ui,
org.eclipse.core.runtime,
org.eclipse.ui.workbench,
org.eclipse.ui.ide;bundle-version="3.13.1",
org.eclipse.core.resources,
org.eclipse.jdt;bundle-version="3.13.4",
org.eclipse.jdt.core;bundle-version="3.13.102"
```

在 eclipse 中，wizard 一般用作新建文件、导入、导出的向导程序。

## 新建资源文件的示例

增加扩展点`org.eclipse.ui.newWizards`,并编辑扩展点

```xml
<extension
         point="org.eclipse.ui.newWizards">
      <category
            id="com.leaderli.li.flow.category"
            name="li flow">
      </category>
      <wizard
            category="com.leaderli.li.flow.category"
            class="com.leaderli.li.flow.wizard.CreateFlowWizard"
            icon="icon/flow.png"
            id="com.leaderli.li.flow.createFlowWizard"
            name="flow">
      </wizard>
</extension>
```


则在我们新建文件时，就可以看到如下选项

![eclipse-plugin-develop-tutorial-setup_wizard.png](eclipse-plugin-develop-tutorial-setup_wizard.png)

```java
package com.leaderli.li.flow.wizard;

import java.io.ByteArrayInputStream;

import org.eclipse.core.resources.IFile;
import org.eclipse.core.resources.IFolder;
import org.eclipse.core.resources.IProject;
import org.eclipse.core.resources.IResource;
import org.eclipse.core.runtime.CoreException;
import org.eclipse.core.runtime.IStatus;
import org.eclipse.jdt.core.IJavaProject;
import org.eclipse.jdt.core.JavaConventions;
import org.eclipse.jface.dialogs.IMessageProvider;
import org.eclipse.jface.viewers.IStructuredSelection;
import org.eclipse.jface.wizard.Wizard;
import org.eclipse.jface.wizard.WizardPage;
import org.eclipse.swt.SWT;
import org.eclipse.swt.layout.GridData;
import org.eclipse.swt.layout.GridLayout;
import org.eclipse.swt.widgets.Composite;
import org.eclipse.swt.widgets.Label;
import org.eclipse.swt.widgets.Text;
import org.eclipse.ui.INewWizard;
import org.eclipse.ui.IWorkbench;
import org.eclipse.ui.IWorkbenchPage;
import org.eclipse.ui.PartInitException;
import org.eclipse.ui.PlatformUI;
import org.eclipse.ui.ide.IDE;

public class CreateFlowWizard extends Wizard implements INewWizard {
 IWorkbench workbench;
 IStructuredSelection selection;
 IProject project;
 IFolder flowFolder;
 private static final String FLOW_EXTENSION = "li";
 private static final String FLOW_FLODER = "flow";

 public CreateFlowWizard() {
  setWindowTitle("New Flow");
 }

 /**
  * wizard可能有多个操作页面，当点击next时会调用getNextPage方法来返回下一个操作页面。
  *
  */
 @Override
 public void addPages() {
  project = getProjectFromSelection(selection);
  //未选中项目时，禁止一切操作，并提示相关信息
  if (project == null) {
   addPage(new ErrorMessageWizardPage());
  } else {
   flowFolder = getFlowFolder();
   addPage(new NewFlowWizardPage());
  }
 }
 /**
  * 是否禁用next和previous按钮
  */
 @Override
 public boolean needsPreviousAndNextButtons() {
  return false;
 }

 private IProject getProjectFromSelection(IStructuredSelection selection) {
  IProject result = null;
  if (selection.size() == 1) {
   final Object firstSelectionElement = selection.getFirstElement();
   if (firstSelectionElement instanceof IJavaProject) {
    final IJavaProject javaProject = (IJavaProject) firstSelectionElement;
    result = javaProject.getProject();
   } else if (firstSelectionElement instanceof IResource) {
    IResource resource = (IResource) firstSelectionElement;
    return resource.getProject();
   }
  }
  return result;
 }

 private IFolder getFlowFolder() {
  IFolder flowFolder = project.getFolder(FLOW_FLODER);
  if (!flowFolder.exists()) {
   try {
    flowFolder.create(false, true, null);
   } catch (CoreException e) {
    e.printStackTrace();
   }
  }
  return flowFolder;
 }

 private IFile getFlowFile(String fileName) {
  return flowFolder.getFile(fileName+"."+FLOW_EXTENSION);
 }

 private String getFileContent(String fileName) {
  String content = "<li></li>";
  return content;
 }

 /**
  * 当点击finish按钮时，生成新建的文件
  */
 @Override
 public boolean performFinish() {
   String fileName = getNamePage().flowName.getText();
  IFile file = getFlowFile(fileName);
  try {
   String content = getFileContent(fileName);
   file.create(new ByteArrayInputStream(content.getBytes()), false, null);
  } catch (CoreException e) {
   e.printStackTrace();
  }
  IWorkbenchPage page = PlatformUI.getWorkbench().getActiveWorkbenchWindow().getActivePage();

  try {
   IDE.openEditor(page, file);
  } catch (PartInitException e) {
   e.printStackTrace();
  }
  return true;
 }

 private NewFlowWizardPage getNamePage() {
  return (NewFlowWizardPage) getPage(NewFlowWizardPage.PAGE_NAME);

 }

 @Override
 public void init(IWorkbench workbench, IStructuredSelection selection) {
  this.workbench = workbench;
  this.selection = selection;
 }

 private class ErrorMessageWizardPage extends WizardPage {
  public static final String PAGE_NAME = "ErrorMessageWizardPage";

  protected ErrorMessageWizardPage() {
   super(PAGE_NAME, "Create a new flow", null);
  }

  @Override
  public void createControl(Composite parent) {
   setControl(parent);
   this.setMessage("must select a project", IMessageProvider.ERROR);
   setPageComplete(false);
  }

 }

 private class NewFlowWizardPage extends WizardPage {
  public static final String PAGE_NAME = "NewFlowWizardPage";

  private Text flowName;

  /**
   * 校验当前页面是否操作完毕，进行下一步操作
   */
  private void checkComplete() {
   setPageComplete(false);
   String fileName = flowName.getText();
   if(fileName.length() ==0) {
    this.setMessage("A flow name cannot be empty", IMessageProvider.WARNING);
    return;
   }
   //判断命名是否符合java类名规范
   IStatus iStatus = JavaConventions.validateJavaTypeName(fileName,"1.8","1.8");
   if(iStatus .getSeverity()!=0) {
    this.setMessage(iStatus.getMessage(),IMessageProvider.ERROR);
    return;

   }
   if(getFlowFile(fileName).exists()) {
    this.setMessage("A flow named "+fileName+" already exists", IMessageProvider.ERROR);
    return;
   }else {
    this.setMessage("Create a new flow named "+fileName);
    setPageComplete(true);
   }

  }


  protected NewFlowWizardPage() {
   super(PAGE_NAME, "Create a new flow", null);
  }

  /**
   * 绘制操作页面
   */

  @Override
  public void createControl(Composite parent) {

   Composite composite = new Composite(parent, SWT.NORMAL);
   setControl(composite);
   composite.setLayout(new GridLayout(2, false));

   new Label(composite, SWT.NORMAL).setText("Name:");
   flowName = new Text(composite, SWT.SINGLE | SWT.BORDER);
   flowName.setLayoutData(new GridData(SWT.FILL, SWT.CENTER, true, false));
   flowName.addModifyListener(e -> {
    checkComplete();
   });
   checkComplete();
  }
 }
}

```

实现的 wizard 页面如下

![eclipse-plugin-develop-tutorial-setup_wizard_page.png](eclipse-plugin-develop-tutorial-setup_wizard_page.png)