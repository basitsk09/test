import { Routes, Route, Navigate } from 'react-router-dom';
import { lazy, Suspense } from 'react';
import NotFoundPage from '../common/Pages/NotFoundPage';
import HomePage from '../common/Pages/HomePage';
import Worklist from '../domains/circle/Worklist';
import MiscellaneousWorklist from '../domains/circle/MiscellaneousWorklist';
import PageLayout from '../common/components/layout/PageLayout';
import Schedule9Table from '../domains/circle/maker/components/Schedule9Table';
import Schedule9ATable from '../domains/circle/maker/components/Schedule9ATable';
import YSASupplementary from '../domains/circle/maker/pages/ysa-sup/YSASupplementary';
import PnlSupplementary from '../domains/circle/maker/pages/pnl-sup/PnlSupplementaryMain';
import MOCPosting from '../domains/circle/maker/components/MOC/MOCPosting';
import Schedule9B from '../domains/circle/maker/pages/schedule-9b/Schedule9B';
import DownloadReports from '../domains/circle/DownloadReports';
import Annex2C from '../domains/circle/maker/components/Annex2C';
import ArchiveReportsPage from '../domains/circle/checker/pages/ArchiveReportsPage';
const Schedule9CProvisionTable = lazy(() => import('../domains/circle/maker/components/Schedule9CProvision'));
const Schedule10 = lazy(() => import('../domains/circle/maker/components/Schedule10'));
//import Schedule10 from '../domains/circle/maker/components/Schedule10';
import RejectedMOC from '../domains/circle/maker/components/MOC/RejectedMoc';
import CFSWorklist from '../domains/circle/CFS/CFSWorklist';
import Schedule9CMigration from '../domains/circle/maker/components/Schedule9CMigration';
import IntraGroupLiability from '../domains/circle/CFS/IntraGroupLiability';
import SC9Supplementary from '../domains/circle/maker/components/SC9Supplementary';
import DownloadArchiveReports from '../domains/circle/CFS/DownloadArchiveReports';
import CircleMakerWorklist from '../domains/circle/maker/CircleMakerWorklist';
import DICGCReport from '../domains/circle/maker/components/DICGC/DICGCReport';
import RW04 from '../domains/circle/maker/components/RW04';
import RW05 from '../domains/circle/maker/components/RW05';
import WriteOff from '../domains/circle/maker/components/Writeoff';

export function Loading() {
  return (
    <p>
      <i>Loading...</i>
    </p>
  );
}

const CircleMakerRouter = () => {
  return (
    <Routes>
      <Route index element={<Navigate to="home" />} />
      <Route path="home" element={<HomePage />} />
      {/* <Route index element={<Navigate to="Write-Off" />} />
      <Route path="Write-Off" element={<WriteOff />} /> */}

      <Route
        path="write-off"
        element={
          <PageLayout heading={'Write-Off'}>
            <WriteOff />
          </PageLayout>
        }
      />

      <Route path="cfs-worklist">
        <Route
          index
          element={
            <PageLayout heading={'CFS Worklist'}>
              <CFSWorklist />
            </PageLayout>
          }
        />

        <Route
          path="intra-group-liablity"
          element={
            <PageLayout heading={'Intra Group Liability'}>
              <IntraGroupLiability />
            </PageLayout>
          }
        />
      </Route>

      <Route path="download-reports">
        <Route index element={<Navigate to="download-archive-reports" />} />
        <Route
          index
          path="download-archive-reports"
          element={
            <PageLayout heading={'Download Archive Reports'}>
              <DownloadArchiveReports />
            </PageLayout>
          }
        />
      </Route>

      <Route path="worklists">
        <Route index element={<Navigate to="worklist" />} />
        <Route path="worklist">
          <Route
            index
            element={
              <PageLayout showCircleFreezed={true} heading={'Balance Sheet Worklist'}>
                <CircleMakerWorklist />
              </PageLayout>
            }
          />
          <Route
            path="pnl-sup"
            element={
              <PageLayout heading={'PNL Supplementary'} /* showCircleFreezed={true} */>
                <PnlSupplementary />
              </PageLayout>
            }
          />
          <Route
            path="sc-09"
            element={
              <PageLayout heading={'Schedule 09'}>
                <Schedule9Table />
              </PageLayout>
            }
          />
          <Route path="sh-02" element={<PageLayout heading={'SH 02'}>{/* <Sh02 /> */}</PageLayout>} />
          <Route
            path="ysa"
            element={
              <PageLayout heading={'YSA Supplementary'}>
                <YSASupplementary />
              </PageLayout>
            }
          />
          <Route path="shc-01" element={<PageLayout heading={'SHC 01'}>{/* <Shc01 /> */}</PageLayout>} />
          <Route
            path="sc-10"
            element={
              <PageLayout heading={'Schedule 10'}>
                <Suspense fallback={<Loading />}>
                  <Schedule10 />
                </Suspense>
              </PageLayout>
            }
          />
          <Route
            path="sc-9c"
            element={
              <PageLayout heading={'Schedule 9C - Provisions'}>
                <Suspense fallback={<Loading />}>
                  <Schedule9CProvisionTable />
                </Suspense>
              </PageLayout>
            }
          />
          <Route
            path="schedule-9b"
            element={
              <PageLayout heading={'Schedule 9B'}>
                <Schedule9B />
              </PageLayout>
            }
          />
          <Route
            path="sc-9a"
            element={
              <PageLayout heading={'Schedule 9A'}>
                <Schedule9ATable />
              </PageLayout>
            }
          />
          <Route
            path="sc-9c-migration"
            element={
              <PageLayout heading={'Schedule 9C Migration'}>
                <Schedule9CMigration />
              </PageLayout>
            }
          />
          <Route
            path="sc-09-supl"
            element={
              <PageLayout heading={'SC 09 Supplementary'}>
                <SC9Supplementary />
              </PageLayout>
            }
          />
          <Route path="qrc-1" element={<PageLayout heading={'QRC 1'}>{/* <Qrc1 /> */}</PageLayout>} />
          <Route path="qrc-4" element={<PageLayout heading={'QRC 4'}>{/* <Qrc4 /> */}</PageLayout>} />
          <Route path="qrc-16" element={<PageLayout heading={'QRC 16'}>{/* <Qrc16 /> */}</PageLayout>} />
        </Route>

        <Route path="miscellaneous-worklist">
          <Route
            index
            element={
              <PageLayout /* showCircleFreezed={false} */ heading={'Miscellaneous Worklist'}>
                <MiscellaneousWorklist />
              </PageLayout>
            }
          />
          <Route
            path="dicgc"
            element={
              <PageLayout heading={'DI & CGC'}>
                <DICGCReport />
              </PageLayout>
            }
          />
          <Route
            path="annex-2c"
            element={
              <PageLayout heading={'Annexure - 2C'}>
                <Annex2C />
              </PageLayout>
            }
          />
          <Route
            path="RW-04"
            element={
              <PageLayout heading={'RW-04'}>
                <RW04 />
              </PageLayout>
            }
          />
          <Route
            path="RW-05"
            element={
              <PageLayout heading={'RW-05'}>
                <RW05 />
              </PageLayout>
            }
          />
        </Route>
      </Route>

      <Route path="download">
        <Route index element={<Navigate to="reports" />} />
        <Route
          path="reports"
          element={
            <PageLayout heading={'Download Reports'}>
              <DownloadReports />
            </PageLayout>
          }
        />
        <Route
          path="archive-reports"
          element={
            <PageLayout heading={'Download Archive Reports'}>
              <ArchiveReportsPage />
            </PageLayout>
          }
        />
      </Route>

      <Route path="moc">
        <Route index element={<Navigate to="moc-posting" />} />
        <Route
          path="moc-posting"
          element={
            <PageLayout showCircleFreezed={true} heading={'MOC'}>
              <MOCPosting />
            </PageLayout>
          }
        />
        <Route
          path="rejected-mocs"
          element={
            <PageLayout showCircleFreezed={true} heading={'Rejected MOCs'}>
              <RejectedMOC />
            </PageLayout>
          }
        />
      </Route>

      <Route path="*" element={<NotFoundPage />} />
    </Routes>
  );
};

export default CircleMakerRouter;
