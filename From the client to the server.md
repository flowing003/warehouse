  @Override
  public void onFileUpload(VirtualPanel vp, AdminFileTransferInterface ft)
    {
    }

  /**
   * Called whenever a file transfer error occurs.
   */
  @Override
  public void onFileTransferError(VirtualPanel vp,AdminFileTransferInterface ft,String errorMessage)
    {
    vp.messageBox(MB_OK,ICON_WARNING,errorMessage,errorMessageHeader);
    close(vp);
    return;
    }

  /**
   * Called when to cancel/close
   */
  private void close(VirtualPanel vp)
    {
    if ( fileOutputStream!=null )
      {
      try { fileOutputStream.close(); }
      catch(final IOException e2) {}
      fileOutputStream=null;
      }

    if ( !inProgress )
      vsm.stopSession(vp.getPanelSession().getIndex());
    else
      {
      // Get that userWindow!
      final VirtualUserWindowInterface vu=((VirtualCUserWindow)(vp.getControlFromID("FILETRAN"))).getPeer();
      final AdminFileTransferInterface userWindow=(AdminFileTransferInterface)vu;
      userWindow.closeFileTransfer();
      vp.setEnabled("TRANSFER",true);
      vp.setEnabled("CLTBUTT",true);
      vp.setEnabled("SRVBUTT",true);
      vp.setText("PROGRESS","0");
      inProgress=false;
      doCancel=true;
      }
    }

  /**
   * Called when the close button of a panel is pressed,
   * but before any other processing has been done. If this
   * function returns false, the application close object
   * will be called (if defined), otherwise a faked key-press
   * of the Escape key is performed. If these two actions
   * failed (because there is no close object or no button or
   * menu item is connected to the Escape key), then the
   * function <code>onPanelClose</code> is called.
   *
   * @return  true  to cancel all default processing.
   */
  @Override
  public boolean onPanelClosing(VirtualPanel vp)
    {
    if ( inProgress )
      vp.messageBox(MB_OK,ICON_WARNING,
                    "The file transfer was cancelled. The created file is corrupt.",
                    errorMessageHeader);
    doCancel=true;
    close(vp);
    return true;
    }

  /**
   * Called to inform the listener that the panel is destroyed.
   *
   * <p>Closes any currently opened file transfer.
   */
  @Override
  public void onPanelDestroy(VirtualPanel panel)
    {
    if ( fileOutputStream!=null )
      {
      try { fileOutputStream.close(); }
      catch(final IOException e2) {}
      fileOutputStream=null;
      }
    }

  /**
   * Called from onFileDownload when the download is complete
   */
  private void downloadComplete(VirtualPanel vp,AdminFileTransferInterface ft)
    {
    inProgress=false;
    ft.closeFileTransfer();
    if ( fileOutputStream!=null )
      {
      try { fileOutputStream.close(); }
      catch(final Exception e){}
      fileOutputStream=null;
      }
    vp.setText("PROGRESS","100");
    vp.setText("MESSAGE","The file transfer has completed succefully, press 'OK' to continue");
    vp.messageBox(MB_OK,ICON_INFORMATION,"The file transfer has completed succefully, press 'OK' to continue","Transfer complete");
    vp.setEnabled("TRANSFER",true);
    vp.setEnabled("CLTBUTT",true);
    vp.setEnabled("SRVBUTT",true);
    }
}
