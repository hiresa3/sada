let title;

if (isJointTransactionFlow()) {
  title = duplicatePerkOverlay?.comboTitle;
} else {
  const currentSpoldTitle = duplicatePerkOverlay[perkDuplicateOverlay?.currentSpold]?.title;
  if (currentSpoldTitle) {
    title = currentSpoldTitle;
  } else {
    title = duplicatePerkOverlay?.title ?? '';
  }
}
