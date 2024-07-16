let title;

if (isJointTransactionFlow()) {
  title = duplicatePerkOverlay?.comboTitle;
} else {
  const currentSpoIdTitle = duplicatePerkOverlay[perkDuplicateOverlay?.currentSpoId]?.title;
  if (currentSpoIdTitle) {
    title = currentSpoIdTitle;
  } else {
    title = duplicatePerkOverlay?.title ? duplicatePerkOverlay?.title : '';
  }
}
